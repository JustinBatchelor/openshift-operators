# openshift-operators
helm chart to deploy various Red Hat openshift-operators to a given OpenShift cluster

## Operators

### Cert-Manager

example invocation from openshift cluster assuming you have installed the `Red Hat OpenShift GitOps Operator` 

**openshift-operators.yaml**

    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
    name: dev-root-app
    namespace: openshift-gitops
    spec:
    project: default
    source:
        repoURL: https://github.com/JustinBatchelor/openshift-operators
        targetRevision: certmanager
        path: ./
        helm:
        parameters:
            - name: clusterName
            value: "dev"
            - name: clusterDomain
            value: "batchelor.live"
            - name: email
            value: "justinrossbatchelor@gmail.com"
            - name: certManager.enabled
            value: "true"
            - name: certManager.ingressCertificate
            value: "true"        
    destination:
        server: https://kubernetes.default.svc
        namespace: openshift-gitops
    syncPolicy:
        automated:
        prune: true
        selfHeal: true

    oc apply -f openshift-operators.yaml

**note:**

- This helm chart will deploy a `clusterissuer` object when the variable `certManager.enabled` is set to `true`. The `clusterissuer` object expects a secret named `cloudflare-api-token` to exist with the key `api-token` in the `cert-manager` namespace

    ```    
    apiVersion: v1
    kind: Secret
    metadata:
        name: cloudflare-api-token
        namespace: cert-manager
    data:
        api-token: <my base64 encoded cloudflare dns zone api token>
    ```

- The cert manager default installations seem to hang when using cloudflare dns resolution if you do not patch the `CertManager` object. Usually the certificate object will throw an error message saying that `Issuing certificate as Secret does not exist`. To fix this run the following command 

    ```
    $ oc patch certmanagers.operator.openshift.io cluster --type=merge -p '{"spec":{"unsupportedConfigOverrides":{"controller": {"args": ["--dns01-recursive-nameservers=1.1.1.1:53,8.8.8.8:53","--dns01-recursive-nameservers-only"]}}}}'
    ```
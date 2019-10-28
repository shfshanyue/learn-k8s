# 使用 Lets Encrypt

+ [Helm Hub jetstack/cert-manager](https://hub.helm.sh/charts/jetstack/cert-manager)

```
ingressShim.defaultIssuerName=letsencrypt-prod
ingressShim.defaultIssuerKind=ClusterIssuer
```

```shell
$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created

$ helm repo add jetstack https://charts.jetstack.io

# 使用 helm v3 部署
$ helm install cert-manager jetstack/cert-manager --set "ingressShim.defaultIssuerName=letsencrypt-prod,ingressShim.defaultIssuerKind=Issuer"
NAME: cert-manager
LAST DEPLOYED: 2019-10-26 21:27:56.488948248 +0800 CST m=+2.081581159
NAMESPACE: default
STATUS: deployed
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://docs.cert-manager.io/en/latest/reference/issuers.html

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://docs.cert-manager.io/en/latest/reference/ingress-shim.html
```

``` shell
$ kubectl get crd
NAME                                  CREATED AT
certificaterequests.cert-manager.io   2019-10-26T01:16:21Z
certificates.cert-manager.io          2019-10-26T01:16:21Z
challenges.acme.cert-manager.io       2019-10-26T01:16:21Z
clusterissuers.cert-manager.io        2019-10-26T01:16:24Z
issuers.cert-manager.io               2019-10-26T01:16:24Z
orders.acme.cert-manager.io           2019-10-26T01:16:21Z

$ kubectl get pods
NAME                                             READY   STATUS    RESTARTS   AGE
cert-manager-5d8fd69d88-s7dtg                    1/1     Running   0          57s
cert-manager-cainjector-755bbf9c6b-ctkdb         1/1     Running   0          57s
cert-manager-webhook-76954fcbcd-h4hrx            1/1     Running   0          57s
```

## 配置 ACME Issuers

## 配置 Ingress

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: example@shanyue.tech
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

## 参考

+ [Automatically creating Certificates for Ingress resources](https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html)
+ [Setting up ACME Issuers](https://docs.cert-manager.io/en/latest/tasks/issuers/setup-acme/index.html)
+ [Get Automatic HTTPS with Let's Encrypt and Kubernetes Ingress](https://akomljen.com/get-automatic-https-with-lets-encrypt-and-kubernetes-ingress/)

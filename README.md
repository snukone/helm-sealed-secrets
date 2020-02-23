# Sealed Secrets

This chart contains the resources to use [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets).

## Prerequisites

* Kubernetes >= 1.9

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
$ helm install --namespace kube-system --name my-release stable/sealed-secrets
```

The command deploys a controller and [CRD](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) for sealed secrets on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
$ helm delete [--purge] my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Using kubeseal

Install the kubeseal CLI by downloading the binary from [sealed-secrets/releases](https://github.com/bitnami-labs/sealed-secrets/releases).

Fetch the public key by passing the release name and namespace:

```bash
kubeseal --fetch-cert \
--controller-name=my-release \
--controller-namespace=my-release-namespace \
> pub-cert.pem
```

Read about kubeseal usage on [sealed-secrets docs](https://github.com/bitnami-labs/sealed-secrets#usage).

1. Install client-side tool into /usr/local/bin/

GOOS=$(go env GOOS)
GOARCH=$(go env GOARCH)
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.8.1/kubeseal-$GOOS-$GOARCH
sudo install -m 755 kubeseal-$GOOS-$GOARCH /usr/local/bin/kubeseal

2. Create a sealed secret file

# note the use of `--dry-run` - this does not create a secret in your cluster
kubectl create secret generic secret-name --dry-run --from-literal=foo=bar -o [json|yaml] | \
 kubeseal \
 --controller-name=sealed-secrets \
 --controller-namespace=default \
 --format [json|yaml] > mysealedsecret.[json|yaml]

The file mysealedsecret.[json|yaml] is a commitable file.

If you would rather not need access to the cluster to generate the sealed secret you can run

kubeseal \
 --controller-name=sealed-secrets \
 --controller-namespace=default \
 --fetch-cert > mycert.pem

to retrieve the public cert used for encryption and store it locally. You can then run 'kubeseal --cert mycert.pem' instead to use the local cert e.g.

kubectl create secret generic secret-name --dry-run --from-literal=foo=bar -o [json|yaml] | \
kubeseal \
 --controller-name=sealed-secrets \
 --controller-namespace=default \
 --format [json|yaml] --cert mycert.pem > mysealedsecret.[json|yaml]

BEISPIEL:

kubectl create secret generic MEINNAME-sealed-secret --dry-run --from-literal=password=HIERPASSWORD -o yaml | \
 kubeseal \
 --controller-name=sealed-secrets \
 --controller-namespace=default \
 --format yaml > MEINNAME-sealed-secret.yaml

3. Apply the sealed secret

kubectl create -f mysealedsecret.[json|yaml]

Running 'kubectl get secret secret-name -o [json|yaml]' will show the decrypted secret that was generated from the sealed secret.

Both the SealedSecret and generated Secret must have the same name and namespace.


## Configuration

| Parameter | Description | Default |
|----------:|:------------|:--------|
| **rbac.create** | `true` if rbac resources should be created | `true` |
| **rbac.pspEnabled** | `true` if psp resources should be created | `false` |
| **serviceAccount.create** | Whether to create a service account or not | `true` |
| **serviceAccount.name** | The name of the service account to create or use | `"sealed-secrets-controller"` |
| **secretName** | The name of the TLS secret containing the key used to encrypt secrets | `"sealed-secrets-key"` |
| **image.tag** | The `Sealed Secrets` image tag | `v0.8.1` |
| **image.pullPolicy** | The image pull policy for the deployment | `IfNotPresent` |
| **image.repository** | The repository to get the controller image from | `quay.io/bitnami/sealed-secrets-controller` |
| **resources** | CPU/Memory resource requests/limits | `{}` |
| **crd.create** | `true` if crd resources should be created | `true` |
| **crd.keep** | `true` if the sealed secret CRD should be kept when the chart is deleted | `true` |
|**networkPolicy** | Whether to create a network policy that allows access to the service | `false`|
|**securityContext.runAsUser** | Defines under which user the operator Pod and its containers/processes run | `1001`|

- In the case that **serviceAccount.create** is `false` and **rbac.create** is `true` it is expected for a service account with the name **serviceAccount.name** to exist _in the same namespace as this chart_ before installation.
- If **serviceAccount.create** is `true` there cannot be an existing service account with the name **serviceAccount.name**.
- If a secret with name **secretName** does not exist _in the same namespace as this chart_, then on install one will be created. If a secret already exists with this name the keys inside will be used.

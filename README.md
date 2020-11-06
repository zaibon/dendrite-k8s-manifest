# K8S manifest for Dendrite

Deploy a [Dendrite](https://github.com/matrix-org/dendrite) monolith server on a K8S cluster.

## Preparation

1. The manifest deploy dendrite into a namespace called `dendrite`. First thing is to create the namspace:

``` shell
kubectl create namespace dendrite
```

2. Generate the Dendrite server signing key

Compile the generate-keys tool from https://github.com/matrix-org/dendrite/tree/master/cmd/generate-keys and generate the server private key:

``` shell
generate-keys -private-key matrix_key.pem
```

3. Create a secret from the private key generated in the previous step

``` shell
kubectl -n dendrite create secret generic dendrite-cert --from-file=matrix_key.pem
```

4. Configure your server name in the dendrite configuration file

Edit the files `dendrite-cm.yaml` and `dendrite-cm.yaml` by replacing `<server_name>` with your actual server name.

5. Deploy dendrite

``` shell
kubectl apply -f dendrite-cm.yaml,dendrite-deployment.yaml,dendrite-pvc.yaml,dendrite-svc.yaml
```

6. Create an ingress to expose the server over your domain

This ingress used [cert-manager](https://cert-manager.io/) to generate TLS certificate automatically from Lets'Encrypt. Make sure you have cert-manager properly installed and functionning in your cluster before deploying the ingress.

Once cert-manager is properly working and your domain point to one of your K8S node that has a public address, deploy the ingress:

``` shell
kubectl apply -f dendrite-ingress.yaml
```

7. Configure the SRV record to configure the federation:

This example expect a SRV record properly configured in order to make the federation work. Checkout the documentation to know how to configure it: https://matrix.org/docs/spec/server_server/latest#resolving-server-names

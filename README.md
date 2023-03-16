# Kubernetes Webhook Certificate Manager

This project is an open source fork of the [newrelic/k8s-webhook-cert-manager](https://github.com/newrelic/k8s-webhook-cert-manager) project, intended to generate a self-signed certificate suitable for use with any Kubernetes Mutating or Validating Webhook.

To be able to execute the script in a Kubernetes cluster, it's released as a Docker image which can be executed as a Kubernetes Job. 

The Docker image is intended to be executed as a Kubernetes Job to perform the following tasks:

- Generate a server key.
- Delete any previous CSR (certificate signing request) for this key if one exists.
- Generate a CSR.
- Submit the CSR for approval via `kubectl`
- The server's certificate is fetched from the CSR and then encoded.
- A Kubernetes secret of type `tls` is created with the generated server certificate and key.
- Fetches the Kubernetes Extension API server's CA bundle and patches the mutating webhook configuration for your webhook server with the CA bundle. 

The Kubernetes Extension API server will use the CA bundle when calling your webhook and validating its certificate. If you wish to learn more about TLS certificate management inside Kubernetes, check out the official documentation for [Managing TLS Certificate in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#create-a-certificate-signing-request-object-to-send-to-the-kubernetes-api).

## Usage example

The script expects multiple mandatory arguments. This is an example:

``` sh
./generate_certificate.sh --service ${WEBHOOK_SERVICE_NAME} --webhook
${WEBHOOK_NAME} --secret ${SECRET_NAME} --namespace ${WEBHOOK_NAMESPACE} 
```

The Docker image can then be used in a Kubernetes job to generate the self-signed certificate:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: webhook-cert-setup
  namespace: default
spec:
  template:
    spec:
      serviceAccountName: webhook-cert-sa
      containers:
      - name: webhook-cert-setup
        # This is a minimal kubectl image based on Alpine Linux that signs certificates using the k8s extension api server
        image: cribl/k8s-webhook-cert-manager:latest
        command: ["./generate_certificate.sh"]
        args:
          - "--service"
          - "scope"
          - "--webhook"
          - "scopecd.appscope.io"
          - "--secret"
          - "scope-secret"
          - "--namespace"
          - "appscope"
      restartPolicy: OnFailure
  backoffLimit: 3
```


## Development setup

This script is designed to run within Kubernetes clusters. For development purposes, we recommend using Minikube.

## License

This project is licensed under the [Apache 2.0](http://apache.org/licenses/LICENSE-2.0.txt) License.

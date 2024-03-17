To check if cert-manager is configured properly and has issued a certificate, you can follow these steps:

### 1. **Check cert-manager Installation**

First, ensure cert-manager is installed in your cluster. You can check if the cert-manager pods are up and running with:

```sh
kubectl get pods --namespace cert-manager
```

You should see the cert-manager, cert-manager-cainjector, and cert-manager-webhook pods running.

### 2. **List Certificates**

List all Certificate resources managed by cert-manager across all namespaces to see if your certificate exists:

```sh
kubectl get certificates --all-namespaces
```

Find the certificate you're interested in from the list. Note its name and the namespace it's in.

### 3. **Describe the Certificate**

To get detailed information about a specific certificate, including its issuance status, use:

```sh
kubectl describe certificate <certificate-name> --namespace <namespace>
```

Replace `<certificate-name>` with the name of your certificate and `<namespace>` with the namespace where the certificate is located. Check the `Status` section to see if the certificate has been successfully issued. A `Ready` condition with a status of `True` indicates the certificate is valid and has been successfully issued.

### 4. **Check Certificate Secret**

cert-manager stores issued certificates as Kubernetes secrets. The Certificate resource should specify the secret name where the certificate and key are stored. Check this secret with:

```sh
kubectl describe secret <secret-name> --namespace <namespace>
```

Replace `<secret-name>` with the name of the secret associated with your certificate. Look for `tls.crt` and `tls.key` data keys; their presence indicates that the certificate and key are stored in this secret.

### 5. **Inspect Certificate Details**

Optionally, you can inspect the details of the certificate directly from the secret to verify its validity, issuer, and other information. First, extract the certificate data:

```sh
kubectl get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.tls\.crt}' | base64 --decode > cert.crt
```

Then, use OpenSSL to inspect the certificate:

```sh
openssl x509 -in cert.crt -text -noout
```

This command prints the certificate's details, allowing you to verify the issuer, the validity period, and other relevant information.

### 6. **Check the Issuer or ClusterIssuer**

It's also a good idea to check the Issuer or ClusterIssuer resource that issued the certificate to ensure it's configured correctly:

- For an Issuer:

    ```sh
    kubectl describe issuer <issuer-name> --namespace <namespace>
    ```

- For a ClusterIssuer:

    ```sh
    kubectl describe clusterissuer <clusterissuer-name>
    ```

Replace `<issuer-name>` or `<clusterissuer-name>` with the name of your Issuer or ClusterIssuer. The status section will provide details on the Issuer's or ClusterIssuer's configuration and operational status.

By following these steps, you can verify that cert-manager is configured correctly in your Kubernetes cluster and has successfully issued a certificate.

*************************
Given the information provided about the `ethereum` certificate in the `ethereum-deployment` namespace, here are the specific commands tailored for your certificate and its related resources:

### Describe the Certificate

To get detailed information about the `ethereum` certificate:

```sh
kubectl describe certificate ethereum --namespace ethereum-deployment
```

### Describe the Secret

To view details of the Kubernetes secret `ethereum-secret` where the certificate and key are stored:

```sh
kubectl describe secret ethereum-secret --namespace ethereum-deployment
```

### Extract and Inspect the Certificate from the Secret

To extract the certificate to a file named `cert.crt` and then inspect its details using OpenSSL:

1. Extract the certificate:

```sh
kubectl get secret ethereum-secret --namespace ethereum-deployment -o jsonpath='{.data.tls\.crt}' | base64 --decode > cert.crt
```

2. Use OpenSSL to inspect the certificate:

```sh
openssl x509 -in cert.crt -text -noout
```

### Describe the ClusterIssuer

To get detailed information about the `acme-issuer` ClusterIssuer:

```sh
kubectl describe clusterissuer acme-issuer
```

These commands allow you to comprehensively review the certificate's configuration, verify its issuance by cert-manager, inspect the associated Kubernetes secret, and check the configuration of the `ClusterIssuer` used for issuing the certificate.
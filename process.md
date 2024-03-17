************ HARBOR *************
```sh
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```
1. Customize harbor values.yaml file: to provide url and also password
```sh
helm show values oci://registry-1.docker.io/bitnamicharts/harbor > harbor-values.yaml
 ```

2. Install harbor in the harbor namespace using helm charts
```sh
 helm install harbor oci://registry-1.docker.io/bitnamicharts/harbor --values harbor-values.yaml -n harbor --create-namespace
```

3. Get the Harbor URL: As mentioned in the output, you need to wait a bit for the LoadBalancer IP to be available. Use the suggested command to monitor the service status:

```sh
kubectl get svc --namespace harbor -w harbor
```

** Please be patient while the chart is being deployed **

3. Get the Harbor URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 

```sh
kubectl get svc --namespace harbor -w harbor
```

```sh
export SERVICE_IP=$(kubectl get svc --namespace harbor harbor --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
```

    ```sh
    echo "Harbor URL: http://$SERVICE_IP/"
    ```

4. Login with the following credentials to see your Harbor application
Given that your Harbor service now has an EXTERNAL-IP of 212.2.244.30, you can access the Harbor UI using this IP address. Here’s how to proceed:

Accessing Harbor
Your Harbor instance is accessible at http://212.2.244.30/ for HTTP or https://212.2.244.30/ for HTTPS access, assuming you have configured TLS. If you've set up an externalURL with a domain that points to this IP, use the domain URL instead.

Logging into Harbor
To log in to Harbor, use the admin credentials:

Username: admin
Password: Retrieve the admin password by executing the command:

  ```sh
  echo Username: "admin"
  echo Password: $(kubectl get secret --namespace harbor harbor-core-envvars -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 -d)
  ```

Create a project on harbor
check for your docker image
tag the image first
 a. docker tag <image name> ethereum.gm-nig-ltd.tech/k8s_ethereum_bloc/<image name>

b. docker images

c. docker push ethereum.gm-nig-ltd.tech/k8s_ethereum_bloc/<image name>

publish the image to harbor


************************ INGRESS - TLS *******************
Files

a. secret-cloudfare.yaml
b. clusterissuer-acme.yaml
c. cert-production.yaml

Commands:
brew install kubernetes-helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install quickstart ingress-nginx/ingress-nginx
kubectl get svc
k apply -f secret-cloudfare.yaml
k apply -f clusterissuer-acme.yaml
k create ns ethereum-deployment
nslookup ethereum.gm-nig-ltd.tech
k apply -f cert-production.yaml
k describe certificate -n ethereum-deployment
k get certificaterequest -n ethereum-deployment
k describe certificaterequest -n ethereum-deployment

*********** INSTALL CERT MANAGER ************
Prerequisites
Install Helm version 3 or later.
Install a supported version of Kubernetes or OpenShift.
Read Compatibility with Kubernetes Platform Providers if you are using Kubernetes on a cloud platform.
Steps
1. Add the Helm repository
This repository is the only supported source of cert-manager charts. There are some other mirrors and copies across the internet, but those are entirely unofficial and could present a security risk.

Notably, the "Helm stable repository" version of cert-manager is deprecated and should not be used.


helm repo add jetstack https://charts.jetstack.io --force-update
2. Update your local Helm chart repository cache:

helm repo update
3. Install CustomResourceDefinitions
cert-manager requires a number of CRD resources, which can be installed manually using kubectl, or using the installCRDs option when installing the Helm chart. Both options are described below and will achieve the same result but with varying consequences. You should consult the CRD Considerations section below for details on each method.

Option 1: installing CRDs with kubectl
Recommended for production installations


kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.crds.yaml
Option 2: install CRDs as part of the Helm release
Recommended for ease of use & compatibility

To automatically install and manage the CRDs as part of your Helm release, you must add the --set installCRDs=true flag to your Helm installation command.

Uncomment the relevant line in the next steps to enable this.

Note that if you're using a helm version based on Kubernetes v1.18 or below (Helm v3.2), installCRDs will not work with cert-manager v0.16. See the v0.16 upgrade notes for more details.

4. Install cert-manager
To install the cert-manager Helm chart, use the Helm install command as described below.


helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  # --set installCRDs=true
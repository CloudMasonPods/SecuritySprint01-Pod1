### 1. Install Argo CD
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Create an Ingress Resource

Create a file named `argo-cd-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd
  annotations:
    kubernetes.io/ingress.class: nginx   # Adjust if you are using a different ingress controller
    cert-manager.io/issuer: <cert-manager-issuer-name>  # Replace with your Cert-Manager issuer name
spec:
  tls:
    - hosts:
        - argocd.gm-nig-ltd.tech  # Your desired DNS name
      secretName: argocd-tls   # Secret name to store TLS certificate
  rules:
    - host: argocd.example.com   # Your desired DNS name
      http:
        paths:
          - pathType: Prefix
            path: /argo-cd
            backend:
              service:
                name: argocd-server
                port:
                  number: 443
```

```bash
kubectl apply -f argo-cd-ingress.yaml
```

### 3. Generate Secret
1. **Generate TLS Certificate**:
   You can generate a TLS certificate using tools like Let's Encrypt, Certbot, OpenSSL, or your preferred certificate authority. The certificate should be issued for the desired DNS hostname (`argo.gm-nig-ltd.tech` in your case).

2. **Create Kubernetes Secret**:
   Once you have the TLS certificate and private key, create a Kubernetes Secret to store them. You can create the Secret using the `kubectl create secret tls` command, providing the certificate and key files as inputs:

   ```bash
   kubectl create secret tls argocd-tls --cert=/Users/simplymenah/Desktop/SecuritySprint01-Pod1/argocd/tls.crt --key=/Users/simplymenah/Desktop/SecuritySprint01-Pod1/argocd/tls.key -n argocd
   ```

   Replace `path/to/tls.crt` and `path/to/tls.key` with the paths to your TLS certificate and private key files, respectively. The `argocd-tls` Secret will be created in the `argocd` namespace.

### 3. Configure DNS

Update your DNS configuration to point `argocd.gm-nig-ltd.tech` (or your chosen DNS name) to the external IP address associated with the Ingress.

### 4. Access Argo CD

Once DNS propagation is complete, you should be able to access Argo CD using `https://argocd.gm-nig-ltd.tech/argo-cd`.

### 5. Login to Argo CD

Retrieve the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d && echo
```
 
Login to Argo CD using the CLI:

```bash
argocd login argocd.example.com
```

Change the password using the command:

```bash
argocd account update-password
```
### 6. Create the ns you want argocd to monitor
```
kubectl create ns eth
```

### 7. Create the secret needed for the deployment/resources  argocd will create to pull from our private container registry
```
 kubectl create secret docker-registry secretname -n namespace \             
--docker-server=private container registry \
--docker-username=username \                                                                               
--docker-password=password

```
### 8. Create the argocd application resource that helps monitor the desired resources
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ethapp
  namespace: argocd
spec:
  destination:
    namespace: eth #namespace to create
    server: 'https://kubernetes.default.svc'
  source:
    paths:  # Specify the required paths to track
      - ethereum-deployment/ethereum-subchart  # Path to the helm ethereum-subchart folder this contains instruction for our deployment
      - ethereum-deployment/ethereum-ingress  # Path to the ethereum-ingress folder we ant to track for the deployments
    repoURL: 'https://github.com/CloudMasonPods/SecuritySprint01-Pod1.git'
    targetRevision: HEAD
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=false #command to create namespace set to false because namespace exist

```
### 9. Ensure the helm charts, kubernetes manifest reside in the path specified above in the git repo 
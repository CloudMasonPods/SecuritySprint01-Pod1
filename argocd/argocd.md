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
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    # If you encounter a redirect loop or are getting a 307 response code
    # then you need to force the nginx ingress to connect to the backend using HTTPS.
    #
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-server-tls # as expected by argocd-server
```

```bash
kubectl apply -f argo-cd-ingress.yaml
```

### 3. Generate Secret
1. **Generate TLS Certificate**:
   You can generate a TLS certificate using tools like Let's Encrypt, Certbot, OpenSSL, or your preferred certificate authority. The certificate should be issued for the desired DNS hostname (`argo.gm-nig-ltd.tech` in your case).

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
### 8. In the deployment part of the value.yaml specify the secret required to pull from our private registry and add the path to the private registry
```
image: "ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app:latest" #image path
imagePullSecrets:
        - name: regcred

```

### 9. Create the argocd application resource that helps monitor the desired resources
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
    path: ethereum-deployment/ethereum-subchart # Specify the required path to track in this case the folder with subchart and ingress
    repoURL: 'https://github.com/CloudMasonPods/SecuritySprint01-Pod1.git'
    targetRevision: HEAD
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true # Automatically create namespace if it doesn't exist

```
### 10. Ensure the helm charts, kubernetes manifest reside in the path specified above in the git repo 
Nb: the helm file should contain value.yanl(here we specify the namespace we want resources to be created) templates.yaml(here we refrence the namespace from value.yaml in all tenplates): templates can be retrieved using the command (helm template helmrepo) and the charts.yaml
```
 helm template ethapp ethereum-helm-charts/ethstats -f /Users/Moyo/Desktop/secure-sc/SecuritySprint01-Pod1/ethereum-deployment/ethereum-subchart/values.yaml > path to values.yaml templates/template.yaml
```
### 11. we can check from the argo cli the state of our app
```
argocd app list
```
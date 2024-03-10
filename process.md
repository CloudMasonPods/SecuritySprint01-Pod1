************ HARBOR *************

1. Customize harbor values.yaml file: to provide url and also password
```sh
helm show values
 oci://registry-1.docker.io/bitnamicharts/harbor > harbor-values.yaml
 ```

2. Install harbor in the harbor namespace using helm charts
```sh
 helm install harbor oci://registry-1.docker.io/bitnamicharts/harbor --values har
bor-values.yaml -n harbor --create-namespace
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



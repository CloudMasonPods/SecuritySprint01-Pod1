### About tekton

Tekton is a powerful and flexible framework for creating continuous integration (CI) and continuous delivery (CD) systems. To have a complete CI setup with Tekton, you need several building blocks:

1. Pipeline: Pipelines in Tekton are a series of tasks that define a CI/CD workflow. They outline the steps your code needs to go through from source code to deployment. Pipelines are defined using YAML manifests.


2. PipelineResource: PipelineResources in Tekton represent external resources that pipelines interact with during their execution. They abstract inputs or outputs of pipeline stages, such as source code repositories, Docker images, or configuration files. PipelineResources define the type, parameters, and name of the associated external resources, enabling flexibility and reusability in CI/CD workflows.

3. Tasks: Tasks are the smallest building blocks in Tekton. They represent a single unit of work in a pipeline, such as building code, running tests, or deploying applications. Tasks are reusable and can be combined to form more complex pipelines.

4. Resources: Resources represent the inputs and outputs of tasks in Tekton pipelines. These could be Git repositories, Docker images, or any other external artifact needed in the CI/CD process.

5. Workspaces: Workspaces are used to share files between tasks within a pipeline. They provide a way to pass data, such as build artifacts or configuration files, between different stages of the pipeline.

6. Trigger: Tekton Trigger allows you to automatically start pipelines in response to events, such as code commits or image updates. This enables you to create fully automated CI/CD workflows that respond to changes in your codebase or infrastructure.

7. Interceptors and Extensions: Tekton Interceptors and Extensions provide advanced capabilities for customizing and extending Tekton pipelines. Interceptors allow you to intercept and modify pipeline execution, while extensions enable you to add new features and integrations to your CI/CD workflows.

8. Dashboard and CLI: Tekton provides a dashboard and command-line interface (CLI) for managing and monitoring pipelines. These tools allow you to view pipeline status, logs, and other relevant information, as well as interact with and control your CI/CD processes.



### Installing tekton on cluster
```
export INGRESS_HOST=$(kubectl \
    --namespace ingress-nginx \
    get svc ingress-nginx-controller \
    --output jsonpath="{.status.loadBalancer.ingress[0].ip}")
```
### Check the ingress host to see if succesfuly exported ip
```
echo $INGRESS_HOST
```
### Install `tkn` by following the instructions from the *Set up the CLI* section in https://tekton.dev/docs/getting-started/
```
git clone https://github.com/vfarcic/tekton-demo.git

cd tekton-demo

# pipeline
kubectl apply \
    --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml 

#install triggers
kubectl apply \
    --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml

#install dashboard
kubectl apply \
    --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```
Or install using helm.


### Creating All required Tekton resources(docker or harbor), cosign, trivy etc 
1. Create a namespace for the Tekton resources
```
kubcetl create namespace tekton-cicd
```

2. create service account for pushing and signing image
```
- k create sa -n tekton-cicd pipeline-sign
```
3. create a secret for container registry credentials(harbor)
nb: secret of type docker-registry is used to create secret for all registry types.
secret type: docker-registry
secret name: registry-secret
```
kubectl create secret docker-registry regcred \
  --docker-server=https://ethereum.gm-nig-ltd.tech\
  --docker-username=admin\
  --docker-password=<registry-password> \
  --namespace=tekton-cicd

```

4. Patch the servicee account to enable imagePullSecrets to authenticate with container image registries when pulling images during deployment or other operation
```
kubectl patch serviceaccount pipeline-sign -n tekton-cicd  -p '{"imagePullSecrets": [{"name": "registrycredentials”}]}

```

5. Patch the service account to add the docker registry secret created above to the list of secrets associated with the service account (Now, even Chains also has push permissions for any TaskRuns running under the service account)
```
kubectl patch serviceaccount pipeline-sign -n tekton-cicd  -p '{"secrets": [{"name": "registrycredentials"}]}'
```

6. Create Tekton tasks
    6a. First we create the task for the git clone

    ```
      # Definition of a Tekton Task named git-clone
          apiVersion: tekton.dev/v1beta1
          kind: Task
          metadata:
            name: git-clone # Name of the task
            namespace: tekton-cicd # Namespace where the task is located
          spec:
            params: # Parameters required by the task
              - name: url
                type: string
              - name: revision
                type: string
                default: main # Default value for the revision parameter if not provided
            results: # Results produced by the task
              - description: The precise commit SHA fetched by this task
                name: commit
            workspaces: # Workspaces for storing and sharing data between steps
              - name: source
                description: The git repo will be cloned onto the volume backing this workspace
            steps: # List of steps to be executed in the task
              - name: clone # Step to clone the Git repository
                image: alpine/git # Docker image to use for the step
                script: | # Script to execute in the step
                  # Check if the repository was previously cloned, if yes, delete the repo and clone a new one
                  if [ -d "$(workspaces.source.path)/repo" ]; then
                    rm -rf "$(workspaces.source.path)/repo" || exit 1
                  fi
                  # Clone git repo into the source workspace
                  git clone $(params.url) $(workspaces.source.path)/repo
                  cd $(workspaces.source.path)/repo
                  # Save Git Commit's Short Sha and push it to the results
                  RESULT_SHA="$(git rev-parse HEAD)"
                  printf "%s" "${RESULT_SHA}" > "$(results.commit.path)"
                  git checkout $(params.revision) # Checkout the specified revision

          
    ```
    6b  create task to check if dockerfile exist in the path
      ```
      # task-check-dockerfile.yaml
      apiVersion: tekton.dev/v1beta1
      kind: Task
      metadata:
        name: check-dockerfile
        namespace: tekton-cicd
      spec:
        params: []
        workspaces:
        - name: source
          description: The workspace containing the cloned Git repository
        steps:
        - name: check-dockerfile
          image: alpine/git
          script: |
            if [ -f "$(workspaces.source.path)/repo/Dockerfile" ]; then
              echo "Dockerfile exists"
            else
              echo "Dockerfile does not exist"
              exit 1
            fi

      ```
    6c create task to build and push to our container registry using our secret created and also adding the namespace
      ```
      # API version and kind of the resource, indicating it's a Tekton Task
              apiVersion: tekton.dev/v1beta1
              kind: Task
              metadata:
                # Name and namespace of the Task
                name: build-and-push-dockerfile
                namespace: tekton-cicd
              spec:
                # Parameters that can be passed to this Task at runtime
                params:
                  - name: image-name
                    type: string
                    description: The name of the image to build and push
                  - name: registry-url
                    type: string
                    description: The URL of the private Harbor registry
                  - name: TAG
                    default: latest
                    type: string
                results:
                  - description: Digest of the image just built.  
                    name: IMAGE_DIGEST
                  - description: Created image
                    name: IMAGE_URL
                # Workspaces define shared storage between steps in a Task or among Tasks in a Pipeline
                workspaces:
                  - name: source
                    description: The workspace containing the cloned Git repository for building

                # Steps are a series of commands executed in a container as part of the Task
                steps:
                  - name: build-and-push
                    # The container image for this step is the Kaniko executor, used for building and pushing Docker images
                    image: gcr.io/kaniko-project/executor:v1.5.1@sha256:c6166717f7fe0b7da44908c986137ecfeab21f31ec3992f6e128fff8a94be8a5
                    env:
                      # Sets the location where Kaniko expects to find Docker configuration files
                      - name: DOCKER_CONFIG
                        value: /kaniko/.docker
                    command:
                      # The Kaniko executor command and arguments for building and pushing the Docker image
                      - /kaniko/executor
                      - --insecure # Allows pushing to a non-TLS or an untrusted TLS registry
                      - --skip-tls-verify # Skips TLS certificate verification
                      - --dockerfile=/workspace/source/repo/Dockerfile # Specifies the path to the Dockerfile
                      - --context=/workspace/source # Specifies the build context
                      # This line is commented out but would specify the image destination using parameters for the registry URL and image name
                      # - --destination=$(params.registry-url)/eth-project/$(params.image-name)
                      - --destination=$(params.registry-url)/$(params.image-name):$(params.TAG)
                      - --digest-file=$(results.IMAGE_DIGEST.path) # this line is very  important for tekton-chain to view the result from build to sign
                    # Mounts a volume containing Docker configuration files necessary for pushing the image
                    volumeMounts:
                      - name: docker-config # The name of the volume to mount
                        mountPath: /kaniko/.docker # The path in the container where the volume is mounted
                  #step to print the built image uri and write the uri to ..very important for tekton-chain to pick up 
                  - name: write-url
                    image: docker.io/library/bash:5.1.4@sha256:c523c636b722339f41b6a431b44588ab2f762c5de5ec3bd7964420ff982fb1d9
                    script: |
                      set -e
                      image="$(params.registry-url)/$(params.image-name):$(params.TAG)"
                      echo -n "${image}" | tee "$(results.IMAGE_URL.path)"

                # Defines volumes used by this Task
                volumes:
                  - name: docker-config # The name of the volume
                    # This volume is backed by a Kubernetes secret containing Docker configuration for Kaniko
                    secret:
                      secretName: regcred # The name of the Kubernetes secret
                      defaultMode: 256 # Sets the UNIX permissions for the files in the secret
                      items:
                      - key: .dockerconfigjson
                        path: config.json
      ```
  
7. Create the Pipeline that for the gitclone to push task
```
 apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: git-clone-pipeline
  namespace: tekton-cicd
spec:
  params:
    - name: git-url
      type: string
    - name: git-revision
      type: string
      default: main
    - name: image-name
      type: string
      description: The name of the image to build and push
    - name: registry-url
      type: string
      description: The URL of the private Harbor registry
  workspaces:
    - name: shared-data
  tasks:
    - name: fetch-repository
      taskRef:
        name: git-clone
      params:
        - name: url
          value: $(params.git-url)
        - name: revision
          value: $(params.git-revision)
      workspaces:
        - name: source
          workspace: shared-data
    - name: check-dockerfile
      taskRef:
        name: check-dockerfile
      runAfter:
        - fetch-repository
      workspaces:
        - name: source
          workspace: shared-data
    - name: build-and-push-dockerfile
      taskRef:
        name: build-and-push-dockerfile
      runAfter:
        - check-dockerfile
      params:
        - name: image-name
          value: $(params.image-name)
        - name: registry-url
          value: $(params.registry-url)
        - name: TAG
          value: $(tasks.fetch-repository.results.commit)
      workspaces:
        - name: source
          workspace: shared-data
```
8. Create the Pipleine run that triggers the pipeline(here we insert our values). and add the service account to it.
```
# Define a PipelineRun resource to execute a Tekton pipeline
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  generateName: git-clone-pipeline-run # Generate a unique name for the PipelineRun
  namespace: tekton-cicd # Specify the namespace where the PipelineRun will be created
spec:
  pipelineRef:
    name: git-clone-pipeline # Reference the Tekton Pipeline object to execute
  serviceAccountName: pipeline-sign # Assign a service account to the PipelineRun for authentication
  params:
    - name: git-url
      value: "https://github.com/CloudMasonPods/SecuritySprint01-Pod1.git" # Specify the URL of the Git repository to clone
    - name: git-revision
      value: main # Specify the branch or revision of the Git repository
    - name: image-name
      value: my-ethereum-app:latest # Specify the name of the image to build
    - name: registry-url
      value: "ethereum.gm-nig-ltd.tech/eth-project" # Specify the URL of the container registry to push the built image
  workspaces:
    - name: shared-data
      persistentVolumeClaim:
        claimName: tektonpvc # Specify the name of the PersistentVolumeClaim to be used as a workspace



```
9. It is important to note the pvc bound to the namespace , that is where the cloned repo would be stored for the subsequent tasks in our case tektonpvc(important to create pvc and bound to the workspace as hsown above)
```
k get pvc -n tekton-cicd
```


### Tekton Chains
This allows the signing of artifacts built from tekton and also production of provenance(which is an attestation that some entity[builder] in our case tekton produced one or more software artifacts byexcuting some invocationusing some other artifacts as input.this invocation runs a buildconfig which is a record of what was executed9process map showing input and process of building artifacts)

#### To install chains
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/chains/latest/release.yaml
```

tekton chain also assists in signing artifacts such as docker images built with tekton using cosign, kms etc.
by simply using the command
```
cosign generate-key pair k8s://tekton-chains/signing-secrets
```
This helps to create the cosign private key,password and pub as a secret called signing-secrets (tekton chain expects the secret called signing-secrets) in tekton-chain namespace, in our case we have an existing cosign key ,pub and password so we need to migrate this existing data to tekton chain ns
```
kubectl create secret generic signing-secrets \
  --from-file=/Users/Moyo/Desktop/Walter-security/cosign/cosign.key \
  --from-file=/Users/Moyo/Desktop/secure-sc/SecuritySprint01-Pod1/cosign.pub\
  --from-literal=cosign-password='your password' \
  --namespace=tekton-chains

Configuring Tekton Chains
You’ll need to make these changes to the Tekton Chains Config:
```
artifacts.taskrun.format=slsa/v1
artifacts.taskrun.storage=oci
artifacts.oci.storage=oci
transparency.enabled=true
```
To do this we need to run this patch on tekton-chains
```
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.taskrun.format": "slsa/v1"}}'
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.taskrun.storage": "oci"}}'
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.oci.storage": "oci"}}'
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"transparency.enabled": "true"}}'

10. this repository is very important to compare your kaniko task with or otheer task such as buildah
    https://github.com/tektoncd/catalog/blob/main/task/

11. we can verify on the cluster if tekton-chain signed our image using the command
```
tkn pr describe --last -o jsonpath='{.spec.params[?(@.name=="registry-url")].value}/{.spec.params[?(@.name=="image-name")].value}' -n tekton-cicd #this prints the last image url  pushed to harbor 

crane ls #crane ls the printed imagename above lists the image tag, attestation for provenance and signed image 
cosign verify --key k8s://tekton-chains/signing-secrets ethereum.gm-nig-ltd.tech/eth-project/my-ethereum-app(in our case)
```
## Harbor Trivy Trigger
1. Go to the harbor registry UI and navigate to the project of interest
2. After navigate to the configuration tab and enable Vulnerability scanning which Automatically scan images on push to the project registry

## Tekton Trigger with push to our github repo
prerequisite : An ingress-controller installed that can give us an external IP e.g nginx
prerequisite : A github repo where we can create the webhook for the eventlistener(this allows for direct communication between github and tekton).
prerequisite :Tekton pipeline running on your cluster
The flow will be: (EventListener) detects git push event -> t will run the action (Trigger)->TriggerBinding will extract data from the event payload, to be used on TaskRun or PipelineRun(in our case pipelinerun)->TriggerTemplate specifies a blueprint for the resource, such as a TaskRun or PipelineRun(we would refer to our existing )

1. Install Tekton triggers to the cluster
```
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.15.1/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.15.1/interceptors.yaml #installing tekton trigger interceptors.e.g github,gitlab, bitbucket etc git interceptor etc this helps intercept payload sent from git when certain event are triggered by filtering for event such as push events      
```

2. Create a webhook secret to be used in creating webhook on github
```
apiVersion: v1
kind: Secret
metadata:
  name: gitWebhookSecret
  namespace: your namespace
type: Opaque
stringData:
  secretToken: "Desired secret"

```

3. Create the required rbac for the trigger actions(kubectl apply in the desired namespace if namespace not specified in definition)
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-triggers-eth
  namespace: your-namespace-name
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: triggers-eth-eventlistener-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: tekton-triggers-eth
    namespace: my-namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: triggers-eth-eventlistener-clusterbinding
subjects:
  - kind: ServiceAccount
    name: tekton-triggers-eth
    namespace: my-namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
---

```

4. Create eventListener (EventListener processes an incoming request and executes a Trigger)and Trigger(A Trigger decides what to do with the received event) which utilize the github interceptor to listen to push events from github  :
```
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: github-listener
  namespace: tekton-cicd
spec:
  serviceAccountName: tekton-triggers-eth # Service account created earlier on 
  triggers:
    - name: github-listener
      interceptors:
        - ref:
            name: "github"
          params:
            - name: "secretRef"
              value:
                secretName: gitwebhooksecret # the secret we created earlier
                secretKey: secretToken
            - name: "eventTypes"
              value: [ "push" ] # filter only push event
      bindings:
        - ref: github-binding # TriggerBinding
      template:
        ref: github-trigger-template #the  TriggerTemplate
  resources:
    kubernetesResource:
      serviceType: NodePort


```
nb:GitHub Interceptor:The  interceptor we’re running is called github. It’s part of the core interceptors that we installed above. It makes sure that the request:
- has a valid format for GitHub webhooks
- matches a pre-defined secret (that we have set while setting the RBAC above)
- matches the push event type
The github interceptor requires a secret token. This token is set when creating the webhook in GitHub and will be validated by the github interceptor when the request arrives

5. create trigger binding for  validating and modifying the incoming request, we need to extract values from it and bind them to variables that we can later use in our Pipeline
```
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: github-binding
spec:
  params:
    - name: git_revision
      value: $(body.ref) #  Example: refs/heads/main or refs/tags/v3.14.1
```

6. Create Trigger template that refrence the pipelinerun to be trigger everytime there is an event
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: github-trigger-template
  namespace: tekton-cicd
spec:
  params:
    - name: git_revision #the git-revision from the triggerbinding we created earlier
  resourcetemplates:
      # the section below is exactly the same as writing a PipelineRun
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: git-clone-pipeline-run
        namespace: tekton-cicd
      spec:
        pipelineRef:
          name: git-clone-pipeline
        # serviceAccountName: pipeline-sign #add the service account to the pipelinerun
        params:
          - name: git-url
            value: "https://github.com/CloudMasonPods/SecuritySprint01-Pod1.git"
          - name: git-revision
            value: main
          - name: image-name
            value: my-ethereum-app
          - name: registry-url
            value: "ethereum.gm-nig-ltd.tech/eth-project"
            # value: "docker.io/chukwuka1488"
        workspaces:
          - name: shared-data
            persistentVolumeClaim:
              claimName: tektonpvc
        podTemplate:
          imagePullSecrets:
          - name: regcred

7. Create Ingress to be able to send a request to our event listener we need to expose it by creating an Ingress resource and pointing it to our event listener service:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /hooks
            pathType: Exact
            backend:
              service:
                name: el-github-listener #prefix el to the eventlistener name as eventlistener creates a service
                port:
                  number: 8080

```
nb: the service we are referencing here is the service the eventlistener creates which it prefix el to .It will use the port 8080, we can inspect the cluster for the svc :k get svc -n tekton-cicd

8. Adding the webhook to our git repo, and add secret created above 
  - navigate to github and click on settings.
  - In settings select webhook and add webhooks
  - Add Payload URL: Your external IP Address from the Ingress with the /hooks path: http://xxx.x.xxx.xxx/hooks
  - Content type: application/json
  - secret which would match the secret created above (in our case gitWebhookSecret)
  - which events wull be trigger:just the push event


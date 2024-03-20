### About tekton

Tekton is a powerful and flexible framework for creating continuous integration (CI) and continuous delivery (CD) systems. To have a complete CI setup with Tekton, you need several building blocks:

1. Pipeline: Pipelines in Tekton are a series of tasks that define a CI/CD workflow. They outline the steps your code needs to go through from source code to deployment. Pipelines are defined using YAML manifests.


2. PipelineResource: PipelineResources in Tekton represent external resources that pipelines interact with during their execution. They abstract inputs or outputs of pipeline stages, such as source code repositories, Docker images, or configuration files. PipelineResources define the type, parameters, and name of the associated external resources, enabling flexibility and reusability in CI/CD workflows.(For example, if you have a pipeline that needs to build a Dockerfile from source code stored in a Git repository and then push the resulting image to a Docker registry, you might define two PipelineResources: one representing the Git repository and another representing the Docker registry. These resources would then be referenced by the tasks within the pipeline to fetch the source code and push the image, respectively.)

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
2. create a secret for container registry credentials(harbor)
nb: secret of type docker-registry is used to create secret for all registry types.
secret type: docker-registry
secret name: registry-secret
```
kubectl create secret docker-registry registry-secret \
  --docker-server=https://ethereum.gm-nig-ltd.tech\
  --docker-username=admin\
  --docker-password=<registry-password> \
  --namespace=tekton-cicd

```

3. generate a cosign key-pair(this assumes that we have cosign installed locally)
```
cosign generate-key-pair

```

4. Create a generic kubernetes secretw with the Cosign private key:
secret name created : cosign-key
secret type: generic
```
kubectl create secret generic cosign-key \
  --from-file=/Users/Moyo/Desktop/Walter-security/cosign/cosign.key\
  --namespace=tekton-cicd
```

kubectl create secret generic cosign-key \
  --from-file=path to cosign key \
  --namespace=tekton-cicd

5. Create Tekton tasks
    5a. First we create the task for the git clone 

    ```
#task-git-clone.yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: git-clone
  namespace: tekton-cicd
spec:
  params:
  - name: url
    type: string
  - name: revision
    type: string
    default: main
  workspaces:
  - name: output
    description: The git repo will be cloned onto the volume backing this workspace
  steps:
  - name: clone
    image: alpine/git #git repo 
    script: |
      git clone $(params.url) $(workspaces.output.path)/repo
      cd $(workspaces.output.path)/repo
      git checkout $(params.revision)
    
    ```
6. Create the Pipeline  that uses the git-clone task
```
# pipeline-git-clone.yaml
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
  workspaces:
  - name: shared-data
  tasks:
  - name: fetch-repository
    taskRef:  #Referencing the task
      name: git-clone
    params:
    - name: url
      value: $(params.git-url)
    - name: revision
      value: $(params.git-revision)
    workspaces:
    - name: output
      workspace: shared-data

```
7. Create the Pipleine run that triggers the pipeline(here we insert our values)
```
# pipelinerun-git-clone.yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: git-clone-pipeline-run
  namespace: tekton-cicd
spec:
  pipelineRef:
    name: git-clone-pipeline
  params:
  - name: git-url
    value: our-git-repository-url
  - name: git-revision
    value: our-git-revision #branch to checkout:e.g main if not specified default is taken from the task level
  workspaces:
  - name: shared-data
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi

```
8. It is important to not the pvc bound to the namespace as that is where the cloned repo would be stored for the subsequent tasks
```
k get pvc -n tekton-cicd
```
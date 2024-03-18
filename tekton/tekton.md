*** About tekton***
Tekton is a powerful and flexible framework for creating continuous integration (CI) and continuous delivery (CD) systems. To have a complete CI setup with Tekton, you need several building blocks:

1. Pipeline: Pipelines in Tekton are a series of tasks that define a CI/CD workflow. They outline the steps your code needs to go through from source code to deployment. Pipelines are defined using YAML manifests.

2.PipelineResource: PipelineResources in Tekton represent external resources that pipelines interact with during their execution. They abstract inputs or outputs of pipeline stages, such as source code repositories, Docker images, or configuration files. PipelineResources define the type, parameters, and name of the associated external resources, enabling flexibility and reusability in CI/CD workflows.(For example, if you have a pipeline that needs to build a Dockerfile from source code stored in a Git repository and then push the resulting image to a Docker registry, you might define two PipelineResources: one representing the Git repository and another representing the Docker registry. These resources would then be referenced by the tasks within the pipeline to fetch the source code and push the image, respectively.)

2. Tasks: Tasks are the smallest building blocks in Tekton. They represent a single unit of work in a pipeline, such as building code, running tests, or deploying applications. Tasks are reusable and can be combined to form more complex pipelines.

3. Resources: Resources represent the inputs and outputs of tasks in Tekton pipelines. These could be Git repositories, Docker images, or any other external artifact needed in the CI/CD process.

4. Workspaces: Workspaces are used to share files between tasks within a pipeline. They provide a way to pass data, such as build artifacts or configuration files, between different stages of the pipeline.

5. Trigger: Tekton Trigger allows you to automatically start pipelines in response to events, such as code commits or image updates. This enables you to create fully automated CI/CD workflows that respond to changes in your codebase or infrastructure.

6. Interceptors and Extensions: Tekton Interceptors and Extensions provide advanced capabilities for customizing and extending Tekton pipelines. Interceptors allow you to intercept and modify pipeline execution, while extensions enable you to add new features and integrations to your CI/CD workflows.

7. Dashboard and CLI: Tekton provides a dashboard and command-line interface (CLI) for managing and monitoring pipelines. These tools allow you to view pipeline status, logs, and other relevant information, as well as interact with and control your CI/CD processes.




***Installing tekton on cluster***
```
export INGRESS_HOST=$(kubectl \
    --namespace ingress-nginx \
    get svc ingress-nginx-controller \
    --output jsonpath="{.status.loadBalancer.ingress[0].ip}")
```
# Check the ingress host to see if succesfuly exported ip
```
echo $INGRESS_HOST
```
# Install `tkn` by following the instructions from the *Set up the CLI* section in https://tekton.dev/docs/getting-started/
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





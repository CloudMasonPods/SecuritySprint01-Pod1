To set up event triggering in Tekton, you'll need to follow these steps to create EventListeners, TriggerTemplates, and TriggerBindings:

Create TriggerBindings:

- Determine which parts of the event payload you want to use as parameters in your pipelines.
- Create a TriggerBinding YAML file defining the mapping between the event payload and pipeline parameters.
- Specify the necessary fields in the TriggerBinding, such as spec.parameters, spec.results, and spec.secretTemplates.
Here's an example TriggerBinding YAML:


apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: my-trigger-binding
spec:
  params:
    - name: param1
      value: $(body.param1)
    - name: param2
      value: $(body.param2)


Create EventListeners:

- Define the EventListener to receive incoming events and forward them to the appropriate TriggerTemplates.
- Configure the EventListener to specify the type of events it listens for and how it handles incoming requests.
Here's an example EventListener YAML:


apiVersion: triggers.tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: my-event-listener
spec:
  triggers:
    - name: my-trigger
      bindings:
        - my-trigger-binding
      template:
        name: my-trigger-template


Create TriggerTemplates:

- Define the conditions and parameters for triggering a pipeline.
- Specify the pipeline and task to execute when the trigger conditions are met.
Here's an example TriggerTemplate YAML:


apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: my-trigger-template
spec:
  params:
    - name: param1
      description: Parameter 1 description
    - name: param2
      description: Parameter 2 description
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: my-pipeline-run-
      spec:
        pipelineRef:
          name: my-pipeline
        params:
          - name: param1
            value: $(tt.params.param1)
          - name: param2
            value: $(tt.params.param2)


Apply the YAML files:

- Save the TriggerBindings, EventListeners, and TriggerTemplates YAML files.
- Use kubectl apply -f <filename.yaml> to apply each YAML file to your Kubernetes cluster.


Verify the setup:

- Once applied, verify that the TriggerBindings, EventListeners, and TriggerTemplates are created successfully using kubectl get commands.
- Check the logs of the EventListeners to ensure they are receiving events correctly.


Tekton.dev documentation on Event Listeners:
https://tekton.dev/vault/triggers-v0.8.1/eventlisteners/

#####################

examples of YAML files for EventListeners, TriggerTemplates, and TriggerBindings to set up triggers for Push Events and Pull Request Events:

EventListener for Push Events:


apiVersion: tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: push-event-listener
spec:
  serviceAccountName: tekton-triggers
  triggers:
    - name: push-trigger
      interceptors:
        - cel:
            filter: "body.ref == 'refs/heads/main'"
      bindings:
        - name: push-binding
  template:
    name: push-pipeline


TriggerTemplate for Push Events:


apiVersion: tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: push-template
spec:
  params:
    - name: git-url
      description: URL of the GitHub repository
    - name: git-revision
      description: Revision (branch, tag, commit) to build
      default: "main"
  resourcetemplates:
    - apiVersion: tekton.dev/v1alpha1
      kind: PipelineRun
      metadata:
        generateName: push-pipeline-run-
      spec:
        pipelineRef:
          name: your-push-pipeline
        params:
          - name: git-url
            value: $(tt.params.git-url)
          - name: git-revision
            value: $(tt.params.git-revision)


TriggerBinding for Push Events:


apiVersion: tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: push-binding
spec:
  params:
    - name: git-url
      value: $(body.repository.clone_url)
    - name: git-revision
      value: $(body.after)


EventListener for Pull Request Events:


apiVersion: tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: pr-event-listener
spec:
  serviceAccountName: tekton-triggers
  triggers:
    - name: pr-trigger
      interceptors:
        - cel:
            filter: "body.action in ['opened', 'synchronize', 'closed']"
      bindings:
        - name: pr-binding
  template:
    name: pr-pipeline


TriggerTemplate for Pull Request Events:


apiVersion: tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: pr-template
spec:
  params:
    - name: pr-number
      description: Pull request number
  resourcetemplates:
    - apiVersion: tekton.dev/v1alpha1
      kind: PipelineRun
      metadata:
        generateName: pr-pipeline-run-
      spec:
        pipelineRef:
          name: your-pr-pipeline
        params:
          - name: pr-number
            value: $(tt.params.pr-number)


TriggerBinding for Pull Request Events:


apiVersion: tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: pr-binding
spec:
  params:
    - name: pr-number
      value: $(body.number)
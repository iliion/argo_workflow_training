# Workflow Templates
  **Workflow templates** (not to be confused with a template) allow you to create a library of code that can be reused. They're similar to pipelines in Jenkins.

  Workflow templates have a different kind to a workflow, but are otherwise very similar:

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: WorkflowTemplate
  metadata:
    name: hello
  spec:
    entrypoint: main
    templates:
      - name: main
        container:
          image: docker/whalesay
  ```
  
  Change to the ***reuse*** directory.

  Lets create this workflow template:
  `argo template create hello-workflowtemplate.yaml -n argo`

  You can also manage templates using kubectl:
  `kubectl apply -f hello-workflowtemplate.yaml -n argo`

  This allows you to use GitOps to manage your templates.

  To submit a template, you can use the UI or the CLI:
  `argo submit --watch --from workflowtemplate/hello -n argo`

  Check @latest
  `argo get @latest -o yaml -n argo`

  Look for the workflow specification in the output:

  ```
  spec:
    workflowTemplateRef:
      name: hello
  ```
  Note how the specification of the workflow is actually a reference to the template.

  ***Exercise***
  Use the user interface to submit a workflow template.
  Update the workflow template to add some parameters (e.g. to print a message). Use `argo -n argo submit --from` to submit it with different parameters.

# Cron Workflows
  A **cron workflow** is a workflow that runs on a cron schedule:

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: CronWorkflow
  metadata:
    name: hello
  spec:
    schedule: "* * * * *"
    workflowSpec:
      entrypoint: main
      templates:
        - name: main
          container:
            image: docker/whalesay
  ```

  When it should be run is set in the schedule field, in the example every minute.

  Lets created this cron workflow:
  `argo cron create hello-cronworkflow.yaml -n argo`

  You'll need to wait for up to a minute to see the workflow run.

  You can also manage templates using kubectl:
  `kubectl apply -f hello-cronworkflow.yaml -n argo`

  This allows you to use GitOps to manage your templates.

  To submit a template, you can use the UI or the CLI:
  `argo submit --watch --from cronworkflow/hello -n argo`

  Check @latest
  `argo get @latest -o yaml -n argo`




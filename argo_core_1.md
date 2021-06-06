# Install Argo Workflows into a Kubernetes cluster.

  `kubectl create ns argo`
  `kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/quick-start-minimal.yaml`

  The Workflow Controller is responsible for running workflows:
  `kubectl -n argo get deploy workflow-controller`

  Users typically want to process and store data in a workflow,
  for the quick start we use ***MinIO***, which is similar to Amazon S3:
  `kubectl -n argo get deploy minio`

  Finally, the Argo Server provides a user interface and API:
  `kubectl -n argo get deploy argo-server`

  Before we proceed, lets wait (around 1 minute to 2 minutes)
  for our deployments to be available:
  `kubectl -n argo wait deploy --all --for condition=Available --timeout 2m`

  A workflow is defined as a Kubernetes resource. Each workflow consists of one
  or more templates, one of which is defined as the entrypoint.
  Each template can be one of several types, in this example we have one template
  that is a container.

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: Workflow
  metadata:
    name: hello
  spec:
    entrypoint: main # the first template to run in the workflows
    templates:
    - name: main
      container: # this is a container template
        image: docker/whalesay # this image prints "hello world" to the console
  ```

  There are several other types of template.
  Because a workflow is just a Kubernetes resource, you can use kubectl with them.
  Create a workflow:
  `kubectl -n argo apply -f hello-workflow.yaml`

  Then you can wait for it to complete (around 1m):
  `kubectl -n argo wait workflows/hello --for condition=Completed --timeout 2m`

  ## Using the UI
  You can view the user interface by running a port forward:
  `kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &`

  To check this is working correctly, you can curl the info API:
  `curl -kv http://localhost:2746/api/v1/info`

  You should see HTTP/1.1 200 OK.

## Run a workflow

### Run your first workflow from the user interface.
  Open the "Argo Server"
  Paste the following yaml into the editor

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: Workflow
  metadata:
    generateName: hello-world-
  spec:
    entrypoint: main
    templates:
    - name: main
      container:
        image: docker/whalesay
  ```
  
### Download, install, and run the Argo CLI.
  To run workflows, the easiest way is to use the Argo CLI, you can download it as follows:

  ```
  curl -sLO https://github.com/argoproj/argo/releases/download/v3.0.2/argo-linux-amd64.gz
  gunzip argo-linux-amd64.gz
  chmod +x argo-linux-amd64
  mv ./argo-linux-amd64 /usr/local/bin/argo
  ```

  To check it is installed correctly:
  `argo version`

  Let's run a workflow!
  `argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml`
  
  You can list workflows easily:
  `argo list -n argo`

  @latest is an alias for the latest workflow:
  `argo get -n argo @latest`

  And you can view that workflows logs:
  `argo logs -n argo @latest`

  Finally, you can get help:
  `argo --help`


# RBAC - WORKFLOW SERVICE ACCOUNT 

> serviceaccount --> (rolebinding) --> role

List roles
`kubectl -n argo get role`
`kubectl -n argo get role workflow-role`
`kubectl -n argo get role workflow-role -o yaml`

Assign role to serviceaccount
`kubectl -n argo create serviceaccount planetek`
`kubectl -n argo create rolebinding planetek --serviceaccount=argo:planetek --role=workflow-role`

Submit workflow 
`argo -n argo submit --serviceaccount planetek simple_wf.yaml --watch`

Check
`kubectl -n argo get workflow | grep from*`



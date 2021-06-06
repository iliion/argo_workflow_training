# Learn about the different types and different categories of template.
## Templates Types
  There are several types of templates, divided into two different categories.

  The first category defines work to be done. This includes:
  - Container
  - Container Set
  - Data
  - Resource
  - Script

  The second category orchestrate the work:
  - DAG
  - Steps
  - Suspend

  Let's analyze containers and DAG.

  A container templates is the most common type of template, lets look at a complete example:

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: Workflow
  metadata:
    generateName: container-
  spec:
    entrypoint: main
    templates:
    - name: main
      container:
        image: docker/whalesay
        command: [cowsay]         
        args: ["hello world"]
  ```
  
  Change to the ***templates*** directory.

  Let's run the workflow:
  `argo submit --watch container-workflow.yaml -n argo`

  And you can view that workflows logs:
  `argo logs -n argo @latest`

# Learn how to template your templates using template tags.
## Template Tags
  Template tags (also knows as template variables) are a way for you to substitute data into your workflow at runtime. Template tags are delimited by {{ and }} and will be replaced when the template is executed.

  What tags are available to use depends on the template type, and there are a number of global ones you can use, such as {{workflow.name}}, which is replaced by the workflow's name:

  ```
  - name: main
    container:
      image: docker/whalesay
      command: [ cowsay ]
      args: [ "hello {{workflow.name}}" ]
  ```

  Look at the full example:
  `cat template-tag-workflow.yaml`

  Submit this workflow:
  `argo submit --watch template-tag-workflow.yaml -n argo`

  You can see the output by running
  `argo logs -n argo @latest `

## Work Templates
  A *container set* allows you to run multiple containers in a single pod. This is useful when you want the containers to share a common workspace.
  A *data template* allows you get data from storage (e.g. S3).
  A *resource template* allows you to create a Kubernetes resource and wait for it to meet a condition (e.g. successful).
  A *script template* allows you to run a script in a container. This is very similar to a container template, but when you've added a script to it.

  Every type of templates that does work does it by running a pod. So you can use *kubectl* to view these pods:
  `kubectl get pods -l workflows.argoproj.io/workflow -n argo`
  Or by using the Argo CLI:
  `argo list pods -n argo`


  You can identify workflow pods by the workflows.argoproj.io/workflow label.
  Use kubectl describe to describe a workflow pod. What interesting information is contained within the pods labels and annotations?

## DAG Template
  A DAG template is another common type of template, lets look at a complete example:

  ```
  apiVersion: argoproj.io/v1alpha1
  kind: Workflow
  metadata:
    generateName: dag-
  spec:
    entrypoint: main
    templates:
      - name: main
        dag:
          tasks:
            - name: a
              template: whalesay
            - name: b
              template: whalesay
              dependencies:
                - a
      - name: whalesay
        container:
          image: docker/whalesay
          command: [ cowsay ]
          args: [ "hello world" ]
  ```

  In this example, we have two templates:

  The **main** template is our new DAG.
  The **whalesay** template is the same template as in the container example.

  The DAG has two tasks: "a" and "b". Both run the **whalesay** template, but as "b" depends on "a", it won't start until "a" has completed successfully.

  Let's run the workflow:
  `argo submit --watch dag-workflow.yaml -n argo`

  Add a new task named "c" to the DAG. Make it depend on both "a" and "b". See how this looks in the user interface.

## Loops
  The ability to run large parallel processing jobs is one of the key features or Argo Workflows. Lets have a look at using loops to do this.

### With Items
  A DAG allows you to loop over a number of items using **withItems**:
  ```
  dag:
    tasks:
      - name: print-message
        template: whalesay
        arguments:
          parameters:
            - name: message
              value: "{{item}}"
        withItems:
          - "hello world"
          - "goodbye world"
  ```

  In this example, it will execute once for each of the listed items. We can see a template tag here. {{item}} will be replaced with "hello world" and "goodbye world". DAGs execute in parallel, so both tasks will be started at the same time.
  `argo submit --watch with-items-workflow.yaml -n argo`

  Notice how the two items ran at the same time.

### With Sequence
  You can also loop over a sequence of numbers using **withSequence**:

  ```
  dag:
    tasks:
      - name: print-message
        template: whalesay
        arguments:
          parameters:
            - name: message
              value: "{{item}}"
        withSequence:
          count: 5
  ```
  As usual, run it:
  `argo submit --watch with-sequence-workflow.yaml`


  Notice how 5 pods were run at the same time, and that their names have the item value in them, zero-indexed

  ***Exercise***
  Change the with sequence to print the numbers 10 to 20.

## Orchestration Templates
  A **DAG** is an orchestration template. What other types of orchestration template are there?
  A **steps template** allows you to run a series of steps in sequence.
  A **suspend template** allows you to automatically suspend a workflow, e.g. while waiting on manual approval, or while an external system does some work.

  Attention!!!
  None of the templates that orchestrate work run pods. You can check by
  `kubectl get pods -n argo`

## Exit Handler
  If you need to perform a task after something has is finished, you can use an exit handle. Exit handlers are specified using ***onExit***:
  ```
  dag:
    tasks:
      - name: a
        template: whalesay
        onExit: tidy-up
  ```

  They just state the name of the template that should be run on-exit.
  Lets look at a complete example:
  `cat exit-handler-workflow.yaml`

  Run it:
  `argo submit --watch exit-handler-workflow.yaml -n argo`


# Parameters
  One type of input or output is a parameter. Unlike artifacts, these are plain string values, and are useful for most simple cases.

## Input Parameters
  Lets have a look at an example:

  ```
      - name: main
        inputs:
          parameters:
            - name: message
        container:
          image: docker/whalesay
          command: [ cowsay ]
          args: [ "{{inputs.parameters.message}}" ]
  ```

  This template declare that it has one input parameter named ***message***.

  See the complete workflow:
  `cat input-parameters-workflow.yaml`
  See how the workflow itself has arguments.

  Run it:
  `argo submit --watch input-parameters-workflow.yaml`
  `argo submit --watch input-parameters-workflow.yaml -p message='Hi Katacoda!'`

  Check logs:
  `argo logs @latest`


## Output Parameters
  Output parameters can be from a few places, but typically the most versatile is from a file. It this example, the container creates a file with a message in it:

  ```
  - name: whalesay
    container:
      image: docker/whalesay
      command: [sh, -c]
      args: ["echo -n hello world > /tmp/hello_world.txt"]
    outputs:
      parameters:
      - name: hello-param        
        valueFrom:
          path: /tmp/hello_world.txt
  ```

  In DAGs and steps template, you can reference the output from one task, as the input to another task using a template tag:

  ```
  dag:
    tasks:
      - name: generate-parameter
        template: whalesay
      - name: consume-parameter
        template: print-message
        dependencies:
          - generate-parameter
        arguments:
          parameters:
            - name: message
              value: "{{tasks.generate-parameter.outputs.parameters.hello-param}}"
  ```

  See the complete workflow:
  `cat parameters-workflow.yaml`

  Run it:
  `argo submit --watch parameters-workflow.yaml`

  Check logs:
  `argo logs @latest`

# Artifacts
  Artifact is a fancy name for a file that is compressed and stored in S3.

  There are two kind of artifact in Argo:

  An **input artifact** is a file downloaded from storage (e.g. S3) and mounted as a volume within the container.
  An **output artifact** is a file created in the container that is uploaded to storage.

  Artifacts are typically uploaded into a bucket within some kind of storage such as S3. We call that storage an artifact repository. We'll be using ***MinIO*** for this purpose, but you can just imagine it is S3.

## Output Artifact
  Each task within a workflow can produce output artifacts. To specify an output artifact, you must include outputs in the manifest. Each output artifact declares:

  - The path within the container where it can be found.
  - A name so that it can be referred to.

  ```
  - name: save-message
    container:
      image: docker/whalesay
      command: [ sh, -c ]
      args: [ "cowsay hello world > /tmp/hello_world.txt" ]
    outputs:
      artifacts:
        - name: hello-art
          path: /tmp/hello_world.txt
  ```

  When the container completes, the file is copied out of it, compressed, and stored.

  Files can also be directories, so when a directory is found all the files are compressed into an archive and stored.

## Input Artifact
  To declare an input artifact, you must include inputs in the manifest. Each input artifact must declare:

  - Its name.
  - The path where it should be created.

  ```
  - name: print-message
    inputs:
      artifacts:
        - name: message
          path: /tmp/message
    container:
      image: docker/whalesay
      command: [ sh, -c ]
      args: [ "cat /tmp/message" ]
  ```

  If the artifact was a compressed directory, it will be uncompressed and unpacked into the path.

## Inputs and Outputs
  You can't use inputs and output is isolation, you need to combine them together using either a steps or a DAG template:

  ```
  - name: main
    dag:
      tasks:
        - name: generate-artifact
          template: save-message
        - name: consume-artifact
          template: print-message
          dependencies:
            - generate-artifact            
          arguments:
            artifacts:
              - name: message
                from: "{{tasks.generate-artifact.outputs.artifacts.hello-art}}"
  ```

  In the above example arguments is used to declare the value for the artifact input. This uses a template tag. In this example, ***{{tasks.generate-artifact.outputs.artifacts.hello-art}}*** becomes the path of the artifact in the repository.

  The task ***consume-artifact*** must run after ***generate-artifact***, so we use dependencies to declare that relationship.

  Lets see the complete workflow:
  `cat artifacts-workflow.yaml`

  Let's run an example:
  `argo submit --watch artifacts-workflow.yaml`

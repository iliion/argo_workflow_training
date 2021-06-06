# Readme

Node.js web app

## App

The app, dependencies, and Dockerfile are in the `/App` folder.

## Kubernetes YAML files

All Kubernetes YAML manifests are in the `Pods`, `Services`, and `Deployments` folders.

-----------------------------

## K8S installation

Install MicroK8s on Linux

`sudo snap install microk8s --classic`


Set up permissions

`sudo usermod -a -G microk8s vagrant`

`sudo chown -f -R vagrant ~/.kube`


Check the status while Kubernetes starts

`microk8s status --wait-ready`


Turn on the services you want

`microk8s enable dashboard dns registry istio`


Try microk8s enable --help for a list of available services and optional features.

`microk8s disable <name> turns off a service`


Start using Kubernetes

`microk8s kubectl get all --all-namespaces`


If you mainly use MicroK8s you can make our kubectl the default one on your command-line
with alias mkctl="microk8s kubectl". Since it is a standard upstream kubectl, you can also
drive other Kubernetes clusters with it by pointing to the respective kubeconfig file via
the --kubeconfig argument.

Access the Kubernetes dashboard

`microk8s dashboard-proxy`


Start and stop Kubernetes to save battery
Kubernetes is a collection of system services that talk to each other all the time.
If you don’t need them running in the background then you will save battery by stopping them

`microk8s start`

`microk8s stop`


## Docker login to *illion13*

## Create and deploy a Pod

`kubectl apply -f pod.yml`

`kubectl get pods `

kubectl describe pods hello-pod

## Create and deploy a Service

`kubectl apply -f svc-nodeport.yml`

`kubectl get svc`

`kubectl describe svc svc-nodeport`

`curl localhost:31111`

 Check

`kubectl get pods -o wide`

## Deployment

`kubectl apply -f deploy.yml`

`kubectl get pods `

`kubectl get deploy -o wide `

`kubectl describe deploy web `

`curl localhost:31111`

## Demonstrate self-healing

`kubectl get pods `

`kubectl delete pod `

`kubectl get pods `

`curl localhost:31111` OR refresh browser

## Demonstrate scaling

`kubectl apply -f deploy_scale_up.yml `

`kubectl get deploy web --watch `

`kubectl get pods --watch `

`kubectl get pods -o wide`

`kubectl apply -f deploy_scale_down.yml `

`kubectl get deploy web --watch `

`kubectl get pods --watch `

`kubectl get pods -o wide`

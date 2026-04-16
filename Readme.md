# Kubernetes Architecture
<img width="1249" height="616" alt="Screenshot 2026-04-16 184449" src="https://github.com/user-attachments/assets/1c3df423-bf14-4cf4-a86d-aa011c7dbeba" />


# Create a kind-config.yml file


kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.35.1
  - role: worker
    image: kindest/node:v1.35.1
  - role: worker
    image: kindest/node:v1.35.1

## We can create cluster using this command

kind create cluster --config kind-config.yaml --name tws-kind-cluster

## Verify cluster

kubectl get nodes
kubectl cluster-info

# we can create namespace, pods, deployments and many other things using yaml file
(we can refer online documentation of kubernetes for yaml format of namespace, pods, etc.)

## for checking namespaces
kubectl get ns

## for checking pods
kubectl get pods

(if in particular namespace)
kubectl get pods -n <namespace>

## we can go into exec mode for that pod

kubectl exec -it <pod-name> -n <namespace> -- bash

## for debugging

kubectl describe pod/<pod-name> -n <namespace>


## for scaling our deployment
kubectl scale deployment/<deployment-name> -n <namespace> --replicas=5

## for more info for pods
kubectl get pods -n <namespace> -o wide

## we can also change image for deployment
kubectl set image deployment/<dep-name> -n <namespace> <container-name>=version

example:
kubectl set image deployment/nginx-dep -n nginx nginx=nginx:latest

## replicaset is same as deployment but the main difference is that we can roll updates or roll backack update possible in deployment

## daemonset ensures that on each and every worker node atleast one pod will run

## we can see logs of pod 
kubectl logs pod/<pod-name> -n <namespace>
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

# Kubernetes Architecture
<img width="1249" height="616" alt="Screenshot 2026-04-16 184449" src="https://github.com/user-attachments/assets/1c3df423-bf14-4cf4-a86d-aa011c7dbeba" />


# Create a kind-config.yml file

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.35.1
  - role: worker
    image: kindest/node:v1.35.1
  - role: worker
    image: kindest/node:v1.35.1
```
## We can create cluster using this command
```yaml
kind create cluster --config kind-config.yaml --name tws-kind-cluster
```
## Verify cluster

```yaml
kubectl get nodes
kubectl cluster-info
```
# we can create namespace, pods, deployments and many other things using yaml file
(we can refer online documentation of kubernetes for yaml format of namespace, pods, etc.)

## for checking namespaces

```yaml
kubectl get ns
```
## for checking pods

```yaml
kubectl get pods
```

(if in particular namespace)

```yaml
kubectl get pods -n <namespace>
```

## we can go into exec mode for that pod

```yaml
kubectl exec -it <pod-name> -n <namespace> -- bash
```

## for debugging

```yaml
kubectl describe pod/<pod-name> -n <namespace>
```

## for scaling our deployment
```yaml
kubectl scale deployment/<deployment-name> -n <namespace> --replicas=5
```

## for more info for pods
```yaml
kubectl get pods -n <namespace> -o wide
```

## we can also change image for deployment
```yaml
kubectl set image deployment/<dep-name> -n <namespace> <container-name>=version
```
example:
```yaml
kubectl set image deployment/nginx-dep -n nginx nginx=nginx:latest
```
## replicaset is same as deployment but the main difference is that we can roll updates or roll backack update possible in deployment

## daemonset ensures that on each and every worker node atleast one pod will run

## we can see logs of pod 
```yaml
kubectl logs pod/<pod-name> -n <namespace>
```

# Persistent Volumes and Persistent Volume Claims
<img width="887" height="626" alt="image" src="https://github.com/user-attachments/assets/364deeb3-288b-4376-91a3-de07edd35b50" />

## we will create a persistent volume(let suppose 1 Gi) from 30 Gb (host machine space) and later to claim this we will create persistent volume claim so it helps us to store our data even when the pod crashed or deleted as in our deployment.yml if any pod crashed or deleted then we can mount the data of the pod to this persistent volume claim so that when pod deleted we have our data in our mounted path.
## Example in case of nginx (data is available at var/www/html so we mount this path to our PVC so that all our data will be stored in our 1Gi volume)


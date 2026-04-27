<img width="1332" height="954" alt="image" src="https://github.com/user-attachments/assets/ed4edbb3-65d3-4401-af8b-570fcc1c2cfd" /># Kubernetes Architecture
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


## getting everything(pod,deployment,service)
```yaml
kubectl get all -n <namespace>
```

# while creating service

## our cluster is docker container so we have to forward port for this docker container 
```yaml
kubectl port-forward service/<our-service-name> -n <namespace> <Port to be mapped with our system port>:<Port of system> --address=<Address to be expose>

kubectl port-forward service/nginx-service -n nginx 80:80 --adress=0.0.0.0
```

# ingress

<img width="1332" height="954" alt="image" src="https://github.com/user-attachments/assets/95d8b709-1333-4f11-a6c1-b343cf13ca69" />

## Ingress acts to transfer traffic/route to our different services, let say /nginx will go to nginx-service and /app will go to our app so we can route our traffic with the help of ingress.

## we need to apply yaml files for ingress controller
```yaml
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

## then we create ingress.yml file in which we route our services

## Main tricky part - Our ingress is running through ingress-controller(which we installed using that kind command) so we have to expose that ingress-controller

```yaml
kubectl get service -n ingress-nginx
```

## then we will get (ingress-nginx-controller)
```yaml
kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 80:80 --address=0.0.0.0
```


# Statefulset
<img width="1364" height="794" alt="Screenshot 2026-04-21 223141" src="https://github.com/user-attachments/assets/1f8aff08-c05e-4fde-8d01-ebfcca3e859e" />

## It is similar like deployment but in case of deployments state is not maintained for example pods are created with random name (as dep-pod-rvzx or dep-pod-wers) like this and if we delete any of these pod its state will gone so to maintain state Statefulsets are used in case of stateful sets pods are created in sequential manner(stateful-pod-0, stateful-pod-1 ....) like this and if we delete any of this pod then new pod is also created with that same name only which we deleted and its state is also maintained.

## in statefulset.yml file we have written how we can create stateful set and we introduced about env variables also that how we can store our variables in configMap and secret and how we can fetch that in statefulset.yml
## For service we have to specify ClusterIP as None so that it will not accessed by outside


# resources and limits for pod

## we define the resources we require for our pod (cpu: 100m,memory: 128Mi) and limits(cpu: 200m,memory: 256Mi) so that it will ensure cluster stability by not overusing any resources.

# Probes - it is a kind of request to check that pod is working or not
  ## Liveness
  ## Readiness
  ## Startup

  ## in the deployment spec we can specify liveness probes, readiness probes to check our pods is successfully running or not.

# Taints/Tolerance
## Taint - it is a way to tell our kubernetes cluster to not schedule pod on that particular node.
## if any node is tainted then also if we want to schedule pod to it then we add tolerance to it so that pod will be scheduled there

## Way of taint:
```yaml
we can find out nodes by kubectl get nodes

kubectl taint node <our node name> prod=true:NoSchedule

For untaint
kubectl taint node <our node name> prod=true:NoSchedule-
```
## now when pod is creating it will not schedule on that particular taint node

## if we want to tolerate then we have to add this in pod.yml(pod specs)
 ## we have given this key as prod because we used this command for taint
 ## kubectl taint node <our node name> prod=true:NoSchedule
```yaml
spec:
  tolerations:
  - key: "prod"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"

so this will help us to tolerate that node and pod will scheduled on node.
```

# AutoScaling - HPA, VPA, KEDA(Kubernetes Event Driven AutoScaling)
## HPA - Increase the number of replicas (Horizontal Scaling)
## VPA - Increase the resources of the pod(earlier - 100Mi , now - 500Mi CPU) (Vertical Scaling)

## Metrics is used to check the resources of our nodes so if we are using kind then we have to install metrics server

## If we are using Kind Cluster install Metrics Server
```yaml
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## edit the metric server deployment
```yaml
kubectl -n kube-system edit deployment metrics-server
```
## add the security bypass to deployment under container.args
```yaml
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP
```

## Restart the deployment
```yaml
kubectl -n kube-system rollout restart deployment metrics-server
```

## To verify metrics server is running
```yaml
kubectl get pods -n kube-system
kubectl top nodes
```
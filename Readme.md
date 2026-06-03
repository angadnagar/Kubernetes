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

  <img width="1107" height="539" alt="Screenshot 2026-04-27 215044" src="https://github.com/user-attachments/assets/2b711bc1-c18c-41e3-8bcb-4c7f1018a443" />


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

<img width="1632" height="964" alt="Screenshot 2026-04-27 221112" src="https://github.com/user-attachments/assets/8c609ddb-d963-44b4-84c3-4141e62c6ecf" />


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

# For VPA
## we have to download some files from git 
## git clone https://github.com/kubernetes/autoscaler.git

## Run ./hack/vpa-up.sh

# RBAC(Role Based Access Control)
## we have to understand two things for this:
- service account
- user

## service account
## it is on namespace level generally, and we have roles(that we can get/delete pods,deployments... in this namespace) and a role binding(for giving this role to service account)

## user
## it is on cluster level, and we have cluster role and cluster role binding

## kubectl auth whoami, it tells about who is the current user

## if we want to check what access user has
```yaml
kubectl auth can-i get pods
kubectl auth can-i get deployment -n <namespace>
```

## we create service-account.yml(new account on which we provide roles)
```yaml
kubectl auth can-i get pods --as=<service-account-user-name> -n <namespace>
```
## so we should create a role.yml file in which we tell what we can access and role-binding.yml in which we tell which user has what role

# cluster level:

## Setting up the kubernetes cluster
```yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

## creata a dashboard-admin-user.yml

## Get the access token 
```yaml
kubectl -n kubernetes-dashboard create token admin-user
```
## Copy the token for use in the Dashboard login.

## Access the Dashboard
```yaml
kubectl proxy
```

## open the dashboard
```yaml
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

## now use the token from previous step to login and from there in dashboard we can monitor


# custom resource definition
## like pod,deployment these are resources in kubernetes so we can create our own resources and for that we have to define our resource and that is known as custom resource definition
## so we create a definition(devops-crd.yml) then based on this we create a custom resource(devops-cr.yml) so our resource is created.

# helm
## Helm is the package manager for Kubernetes, designed to simplify the creation, packaging, configuration, and deployment of applications to Kubernetes clusters.
## In simple words instead of creating deployments, services,etc for different applications(nginx,apache). helm helps to do this without creating these resources seperately

## helm charts: These are structured packages containing YAML templates and configuration values that define all necessary Kubernetes resources (such as Deployments, Services, and ConfigMaps) required to run an application. 

## script to install helm
```yaml
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```

## For creating helm chart
```yaml
helm create <helm-chart-name>
eg. helm create apache-helm
```

## this is structure how it create each yam file for template and values.
<img width="1025" height="732" alt="Screenshot 2026-05-16 220518" src="https://github.com/user-attachments/assets/2c68b8cc-1e50-43b9-b0ff-ab8352b9e7d5" />


## after making any changed to template and value.yml we package our chart
```yaml
helm package apache-helm/
```

## Now we install our chart
```yaml
helm install <name> <chart name> -n <namespace> --create-namespace

helm install dev-apache apache-helm -n dev-apache --create-namespace

similarly we can make for productiona and all like as we made for dev(dev-apache)
```

## after this we can check by kubectl get pods -n dev-apache our pods/deployment everything created

## Now if we want to change anything then we can change values.yaml and then our Chart.yml and update its appVersion (as we have changed in values so it indicates our file changed)
## Now after this again package, " helm package apache-helm "

## Now we can also upgrade this changes 
```yaml
helm upgrade prd-apache ./apache-helm -n prd-apache
```
## Now we can see let suppose for this new version we changed replicas from 2 to 3 so for this particular namespace we have now 3 replicas and for dev one we have 2.
## we can also rollback our change to previous one
```yaml
helm rollback prd-apache <prev revision number> -n <namespace>

helm rollback prd-apache 1 -n prd-apache
```

## we can also check our installed repos by this
```yaml
helm search repo <repo name>
helm search repo nginx

helm repo list     (to see what repo we added)
```
## we can also add repositories
```yaml
helm repo add <repo name>
helm repo add stable https://charts.helm.sh/stable
```
## for this link which is used for repo install, we can find it from "artifacthub". We can directly install what we want from here.
```yaml
helm install <name for our release> oci://registry-i.docker.io/bitnamicharts/nginx
helm install nginx-helm oci://registry-i.docker.io/bitnamicharts/nginx -n nginx --create-namespace

we can also uninstall with this
helm uninstall <name>
helm uninstall nginx-helm
```


# Init Container vs Side car container
## Init Container is used when some initialization is needed before the main container runs such as initial setup and all
## Side car container works with main container it helps main container 

```yaml
Important commands
kubectl logs <name of pod> -c <container-name>
kubectl logs init-test -c init-container
```

# Service Mesh
## A service mesh is a dedicated infrastructure layer that manages service-to-service communication within a microservices architecture.
## like in zomato there are various services such as for menu, orders, delivery, events and how these services redirect we need "Gateway" and for managing these traffic where to redirect we need Service Mesh

# Istio: Popular tool for service mesh
## refer this when installing istio and setup : https://istio.io/latest/docs/ambient/getting-started/ and https://istio.io/latest/docs/setup/getting-started/

## In istio we have these components:
- ## Envoy: Sidecar container/proxies per microservice to handle ingress/egress traffic between services in the cluster and from services to external services.
- ## Istiod: It is control plane of Istio. It provides service discovery (all this we can find in readme when installing istio)             

# Project - 3 Tier Application Deployment on Kubernetes(Reactjs,Nodejs,MongoDB)
## inside projects folder added k8s manifest files for the chat app, with those file we can deploy and expose our frontend/backend service to external user and with the help of ingress we route different service traffic to different routes.

# Kubernetes Monitoring
## Observability: It is based on 
##  1. Metrics  ->  Monitoring (Prometheus, Grafana, Kibana)
##  2. Logs     ->  Logging (Loki, Promtail)
##  3. Traces   ->  Tracing (Jaegar Opentelemetry)

## with metrics we know (What is happening in our server), with logs we know (Why this is happening), with traces we know (How)

## there is node exporter which exports all worker nodes data to a specific port(let 9100) and inside kube-system namespace there are services of master node(api server, scheduler, etcd, controller manager) all this also needs to be monitored and for this kube-state-metrics is required which monitors the whole cluster

## and all this data from all components (like scheduler, api server) goes to Prometheus and then all this data goes to Visualization(Grafana)

## Now we will setup prometheus and grafana
## We first install helm 
## then we add prometheus-community using helm with this it is added in repo and then we update our repo
```yaml
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

we can get this prometheus-community helm chart link in github

helm repo update
```

## now with helm we install prometheus-stack
```yaml
helm install prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set prometheus.service.type=NodePort

if we want to set something in values.yaml in helm then we can set that using this --set prometheus.service.nodePort this means inside prometheus service set nodePort
```
## after this we can check that all pods are running such as node-exporter, kube-state-metrics and all

## then we can port forward the service for prometheus(on 9090) and grafana(on 3000) to access them

## then for logging into grafana we need admin password so we can get password from secret of grafana service
```yaml
                    (service name)                                  (secrets are generally present in .data)
kubectl get secret prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

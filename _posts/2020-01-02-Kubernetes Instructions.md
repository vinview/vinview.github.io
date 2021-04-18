---
title: "Kubernetes Instructions"
date: 2020-01-02
header:
  teaser: /assets/images/nature.jpg
excerpt: "Some instructions and commands in Kubernetes"
---
**K8S installation instruction:**  
Follow this https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows  
(Download kubectl.exe & copy it to C:\windows\system32)  
Minikube: https://github.com/kubernetes/minikube/releases (install minikube-installer.exe)  
(Download version 1.2.0, from assets section)  
Run: minikube start (Minikube is installed in C:\prog~\kubernetes\minikube)  
Minikube dashboard  
Ctrl+C  
Minikube stop  

**Docker SWARM Vs Kubernetes:**   

**Docker/SWARM:**  
(+) Light weight ("PS")  
(+) portable  
(+) Infra(runtime) services  

(-)Too many IPs  
(-)Master is for orchestration + Ops  
(-)Service availability is not 100% (M+R>SD)  
(-)No elasticity  
(-)Single Point of Failure (Admin)  
(-)Invest on Reachable(s)  
(-)JSON/YAML/SHA256/blrfts  
(-)Orchestrated  

**Kubernetes:**  
(+) Any container orchestration  
(+) Any kernel  
(+) Any CSP/non-CSP  
(+) Open source  
(+) Master is only for orchestration  
(+) one IP - master IP  
(+) UoS(expand)  
(+) Orchestration as a service(self heal)  
(+) AI on orchestration  
(+) scale-> service availability  
(+) Manage change (Image change, Port change, Volume change)  


In Kubernetes, only dynamic ports (no static port, -p)  

**Scaling:**  
Service & Infra
3 axis scaling (x,y,z)  
X: Machine/Infra scaling (VMs)  
Y: Services(with memory, CPUs) scaling (Containers)  
Z: Dependency services scaling  

**Master & Node**  
**Master:**  
	- Notary?(RSA)  
	- Authentication  
	- Valid service?  
	- Service? Etcd?  
	- Servicable?  
	- LB (of node)  
Runs in port no-8443  
kubectl cluster-info  

**Node:**  
VM + Container runtime(default is docker. i.e. dockerd)  
Runs in port no-443  

K8S-> 1 master + 7 nodes  
K3S-> 1 master + 3 nodes  

**Components of Master:**  
Services rigitry (by default, etcd). (What service is running)  
Services gateway (How to reach the service): Default is  Kube DNS, others for ex: nginx, nginx plus  
Service discovery (where is service running): Default Kube API server, others: Stack, Ribbon.. Load balancing is done here to find which node  
Workflow:  
1)Service gateway -> 2)service registry -> 3)Memory & storage provider -> 4)service discovery ->5) Ingress controller  

**Components of Node:**  
Agent: Kubelet… Load balancing is is done here via kube proxy for which Pod  
Proxy: Kube Proxy (IP tables)  
Dockerd  
Logs: Fulentd  
Workflow:  
6)Kubelet->7)kube proxy->8)POD->9)Logs(fluentd)->10)Ingress controller-> 11) service gateway  

**Namespace:** Collection of PODs. Virtual cluster. For logical quota.   
3 default namespaces available:  
1.kube-system- master details  
2.kube-public - Internal cluster details  
3.default  

**CLIs:**  
Kubectl & Kubeadm('minikube' when using minikube)  

**CMD1 terminal(For Ops):**    
Minikube status  
Minikube start  
Minikube ip  
kubectl get nodes  
kubectl get namespaces  
kubectl get namespaces  
kubectl describe namespace  kube-public  
kubectl create ns demo  
kubectl describe namespace demo  
kubectl get pods -n kube-system(kube-system is a name of the namespace for ex:)  
kubectl get pods -> searched in default namespace  
kubectl describe pod etcd-minikube -n kube-system  
kubectl get pod etcd-minikube -n kube-system  
kubectl get pod etcd-minikube -n kube-system -o json  
kubectl get pod etcd-minikube -n kube-system -o yaml  
kubectl run nginx-container1 --image=nginx --port=80 -n demo (POD name cannot be set)  
kubectl get pods -n demo  
kubectl describe pod nginx-container1-7cd546bffb-6627g -n demo  
Note: 10.x.x.x is public IP & 172.x.x.x is private IP  
kubectl exec -it nginx-container1-7cd546bffb-6627g bash -n demo  
(Inside the container/POD machine, you can run commands like 'hostname' & exit)  
kubectl get pods -n demo -o wide  
kubectl run nginx-containererror --image=ngnx --port=80 -n demo (create a pod with a wrong image. POD/Deployment is still created)  
kubectl get pods -n demo  
kubectl delete pod nginx-containererror-6777d4b675-rm6l8 -n demo  
kubectl get pods -n demo (Though we have deleted the container, through self healing another container will be created)  
kubectl get deployment -n demo  
kubectl get pods -n demo  
kubectl delete deployment nginx-containererror -n demo (Now the pod is deleted & container is not re-created)  
kubectl get roles -n kube-system  
kubectl get clusterroles -n kube-system  
kubectl describe deployment nginx-container1 -n demo  
kubectl scale deployment nginx-container1 --replicas=5 -n demo  
(When you scale via deployment, memory is grabbed first & then pods are created. So, if there are any memory issue, deployment itself will not be created. i.e. no half way creation of PODs)  
kubectl get deployment -n demo  
kubectl get pods -n demo  
kubectl delete pod nginx-container-95d6675d8-g57zw -n demo  
kubectl expose deployment nginx-container1 --type=NodePort -n demo  
kubectl get services -n demo (This is service descovery: We are reading the service)  
kubectl describe service nginx-container1 -n demo  
minikube service nginx-container1 -n demo (In real time, we use 'kubeadm') (we are Accessing the service here. Here we donot access via PORT, but via service name)  
minikube service nginx-container1 -n demo --url=true (Just to get the URL)  
kubectl run nginx-cp --image=nginx --port=80 -n demo  
kubectl get deployments -n demo  
kubectl expose deployment nginx-cp --type=ClusterIP -n demo (Here, type is cluster IP. i.e. accessed via cluster IP)  
kubectl get service -n demo  
(Take cluster IP & curl the service URL from 'minikube ssh')  
i.e. 
Minikube ssh  
Sudo -i  
curl http://10.97.75.105  

kubectl run nginx-mp --image=nginx --port=80 --replicas=3 -n demo  
kubectl get deployments -n demo  
kubectl expose deployment nginx-mp --type=LoadBalancer -n demo (Here exposed(or accessed via) to master IP)  
kubectl get services -n demo  
kubectl describe service nginx-mp -n demo  
minikube service nginx-mp -n demo  
kubectl get services --all-namespaces (List all services across all namespaces)  
kubectl delete deployment nginx-mp -n demo (Here, deployment & pod are deleted, but service is not deleted)  

| UseCases:    | Node                              | LoadBalancer          | ClusterIP            |
|--------------|-----------------------------------|-----------------------|----------------------|
| Replicas?    | Point                             | Yes                   | No (not recommended) |
| LB?          | Yes(Node)                         | Kube DNS + Kubelet    | Not there            |
| Access?      | Kubelet                           | Master IP/External IP | Cluster IP           |
| #Estimation? | Node IP                           | 50-60%                | 10-15%               |
| Benefit      | only 25% of services will be here | Stable                | Futuristic           |
| Dynamic Port | Pilot                             | Yes                   | No                   |
| Customize    | Yes                               |                       |                      |  

**Secrets**  
kubectl get secret  
kubectl describe secret default-token-hj8qh (<secret_name)  
kubectl create secret generic db-pass-values --from-literal=user=root --from-literal=password=admin  
kubectl get secret  
kubectl describe secret db-pass-values  
kubectl get secret db-pass-values  -o json  
kubectl create secret generic db-volume --from-file=user.txt --from-file=pwd.txt (create 2 files before hand user.txt & pwd.txt with user name & password resp)  
kubectl get secretes  

**Config Map**  
kubectl get cm  
kubectl get cm -n kube-system  
kubectl describe cm kubeadm-config -n kube-system -> this gives property of cluster  
kubectl create cm language-key --from-literal=language="UK_ENGLISH"  
Note: Object name cannot have _ (for ex: container name, service name, config map etc)  
kubectl get cm  
kubectl describe cm language-key  

kubectl create namespace rollout  
kubectl run nginx-roll --image=nginx --replicas=10 --port=80 -n rollout  
kubectl get deployments -n rollout  
kubectl expose deployments nginx-roll --type=NodePort -n rollout  
minikube service nginx-roll -n rollout  
kubectl describe deployment nginx-roll -n rollout  

HPA(Horizontal POD availability)  
Rollout strategy:  

| MAX | Surge |  Unavailable (25% by default) | Min |
|-----|-------|-------------------------------|-----|
| 10  | 10+3  | 3                             | 6   |

Describe deployment will show these details like Max, Surge etc.  

kubectl set image deployment/nginx-roll nginx-roll=bitnami/nginx -n rollout  
kubectl rollout status deployment/nginx-roll -n rollout  
kubectl get pods -n rollout  
kubectl describe pod nginx-roll-566d564589-6qv4j -n rollout  
kubectl rollout undo deployment/nginx-roll -n rollout  
kubectl rollout history ….. -> to see the rollout history  

In Docker perspective, POD is a paused container.  
i.e.   
Minikube ssh  
Sudo -I  
Docker ps | grep nginx-cp(containername)  

**Dashboard:**    
minikube addons list  
minikube addons enable heapster  
minikube addons enable ingress  
kubectl get pods -n kube-system (Now you can see some more pods are created since we have enabled above 2 addons)  
Minikube dashboard (dasboard URL will openup. Here you can scale the deployment etc from the browser)  
minikube addons open heapster (it will open up dashboard link where you can see the graphs for Pods)  

kubectl run ubuntu-cont --image=ubuntu -> POD without a service  
kubectl get pods -> this will show the POD status as ContainerCreating->CrashLoopBackOff->Completed->ContainerCreating.. It will keep on restarting.. Because default restart policy of container in k8s is 'restart_always'  
kubectl describe pod ubuntu-cont-db87d5bc4-tbck7  

Create PODs from YML with no restart  
YML files asre available at: https://github.com/vinaydhegde/DockerStuff/tree/master/K8S  
kubectl get services -n kube-system -> Get Kube DNS IP  

Kubectl get pods  
Kubectl logs command-demo  
Kubectl get secrets  
kubectl create -f pod1.yml  
kubectl create -f pod2.yml  
Kubectl exec -it secret-envirs-test-pod bash  
Echo $SECRET_USERNAME  
Echo $SECRET_PASSWORD  
Kubectl get secrets  

kubectl create -f pod3.yml  
kubectl exec -it mypod bash  
Cd /etc/vols  
Ls  
Cat user.txt  
Cat pwd.txt  

kubectl describe ns default -> It says no resource quota for default namespace  
In the K8sDay5 git repo, refer file called 'quota'  
Set the quota now;  
Kubectl create -f quota.txt  
Kubectl get quota  
Kubectl get resourcequota pod-demo -o yml  
Kubectl run test-demo --image=nginx --replicas=3 --port=80  
Kubectl get deployments  
Kubectl delete quota pod-demo  
Kubectl get deployments  

**Communication between PODs:**    
For example:  
NS1: has 2 pods (POD1 has 2 containers) & NS2 has 1 pod  
	1. Container1 -> container 2 in POD1: via volume  
	2. POd1->POD2: via service  
	3. POD2->POD3: via namespace.service  

**Volume types:**    
	1. Persistent volume(Master volume)  
	2. POD volume:(Empty dir)  
	3. Local volume(node volume)  


Create 2 containers in one POD:  
kubectl create -f pod5.yml  
kubectl get pods  
kubectl describe pod two-containers  (2 containers are created.. Ngin cont is running & ubuntu is terminated)  
kubectl -exec -it two-containers -c nginx-container bash  
>> hostname (POD name is the hostname)  
Exit  

Ingress tutorial:  
kubectl get pods -n kube-system  
Create a POD & service:  
kubectl create -f pod6.yml  
kubectl create -f pod7.yml  
kubectl create -f pod8.yml  
kubectl get pods  
kubectl get services -> note it's a cluster ip service  
We already have ingreass controller, we need to create ingress router  
kubectl get ing  
http://<localhostip>/crap  
http://<localhostip>/apple  
http://<localhostip>/verizon  

Minikube stop  

**CMD2(for kubernetes Admin):**  
Minikube dashboard  

**CMD3(for admin):**  
Minikube ssh  
Sudo -i  



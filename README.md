# How-To-Use-Nginx-Ingress-As-Loadbalancer-in-Kubernetes
###  Requirement:
We need to put some rewrite or rediect rules of our services. Before we use apache or bigip to to do it .Now in K8S, we plan to mannually install Nginx Ingress to meet the requirement

###  Preparation

* Install nginx ingress via helm
```
helm install stable/nginx-ingress --name my-nginx --set rbac.create=true
```
* You would see below similar output
```
$ kubectl get pod
NAME                                                      READY     STATUS    RESTARTS   AGE
my-nginx-nginx-ingress-controller-c886f578d-zwxd5         1/1       Running   0          10m
my-nginx-nginx-ingress-default-backend-6946f9777c-cnj4r   1/1       Running   0          10m

$ kubectl get svc
NAME                                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
my-nginx-nginx-ingress-controller        LoadBalancer   10.105.94.233    <pending>     80:32155/TCP,443:32325/TCP   10m
my-nginx-nginx-ingress-default-backend   ClusterIP      10.96.219.102    <none>        80/TCP                       10m
```
* So far as OCI balancer type is not supported by default, we update svc loadbalancer type to be NodePort
```
kubectl get svc my-nginx-nginx-ingress-controller -o yaml > my-nginx-nginx-ingress-controller-svc-lb.yaml
cp my-nginx-nginx-ingress-controller-svc-lb.yaml my-nginx-nginx-ingress-controller-svc-nodeport.yaml
vim the my-nginx-nginx-ingress-controller-svc-nodeport.yaml and remove type loadbalancer and use NodePort
kubectl delete -f my-nginx-nginx-ingress-controller-svc-lb.yaml
kubectl create -f my-nginx-nginx-ingress-controller-svc-nodeport.yaml
$ kubectl get svc
NAME                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
my-nginx-nginx-ingress-controller        NodePort    10.105.94.233    <none>        80:32155/TCP,443:32325/TCP   7m
my-nginx-nginx-ingress-default-backend   ClusterIP   10.96.219.102    <none>        80/TCP                       28m
```

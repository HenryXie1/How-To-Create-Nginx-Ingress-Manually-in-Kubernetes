# How-To-Create-Nginx-Ingress-Manually-in-Kubernetes
###  Requirement:
We need to put some rewrite or rediect rules of our services. Before we use apache or bigip to to do it .Now in K8S, we plan to mannually install Nginx Ingress to meet the requirement

###  Preparation

* Install nginx ingress via helm
```
helm install stable/nginx-ingress --name my-nginx --set rbac.create=true
```

# How-To-Use-Nginx-Ingress-To-Rewrite-Url-in-Kubernetes
###  Requirement:
We need to put some rewrite  rules for url of our services. Before we use apache or bigip to to do it .Now in K8S, we plan to mannually install Nginx Ingress to meet the requirement.
Please refer more details in official [github doc of nginx ingress controller](https://github.com/kubernetes/ingress-nginx/tree/master/docs)

###  Preparation

* Install helm. Please refer official [github tutorial](https://github.com/oracle/mysql-operator/blob/master/docs/tutorial.md)
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
###  Start to create rules to meet rewrite requirements
* Create a rule to rewrite /test to /ords. Create rules for mutiple virtual hosts. ie apexsb-lb.oraclecorp.com livesqlsb-lb.oraclecorp.com, yaml is like
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /ords
  name: rewrite-test
  namespace: default
spec:
  rules:
  - host: apexsb-lb.oraclecorp.com
    http:
      paths:
      - backend:
          serviceName: apexords-service
          servicePort: 8888
        path: /test
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/app-root: /ords
  name: apexroot
  namespace: default
spec:
  rules:
  - host: apexsb-lb.oraclecorp.com
    http:
      paths:
      - backend:
          serviceName: apexords-service
          servicePort: 8888
        path: /
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/app-root: /ords/f?p=590:1000
  name: livesqlsbroot
  namespace: default
spec:
  rules:
  - host: livesqlsb-lb.oraclecorp.com
    http:
      paths:
      - backend:
          serviceName: livesqlsb-service
          servicePort: 8888
        path: /
```
* Create or Modify OCI load balancer to add backend node pool with  http port 32155  https port 32325
* Use curl to verify the rewrite is working
```
$ curl -I -k http://apexsb-lb.oraclecorp.com/test
HTTP/1.1 302 Found
Date: Fri, 07 Dec 2018 00:35:35 GMT
Content-Type: text/html;charset=utf-8
Connection: keep-alive
X-Content-Type-Options: nosniff
X-Xss-Protection: 1; mode=block
Cache-Control: no-store
Pragma: no-cache
Expires: Sun, 27 Jul 1997 13:00:00 GMT
Set-Cookie: ORA_WWV_USER_230138572008313=ORA_WWV-P233NLbDmLfEN3vyFlL50kKh; path=/ords; HttpOnly
Location: http://apexsb-lb.oraclecorp.com/ords/f?p=4550:1:597976824567:::::

$ curl -I -k http://apexsb-lb.oraclecorp.com/
HTTP/1.1 302 Moved Temporarily
Date: Thu, 06 Dec 2018 06:10:21 GMT
Content-Type: text/html
Content-Length: 145
Connection: keep-alive
Location: http://apexsb-lb.oraclecorp.com/ords
```

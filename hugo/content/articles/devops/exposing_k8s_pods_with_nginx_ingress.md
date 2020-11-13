---
title: "Exposing Pods in K8s with Nginx Ingress"
date: 2020-11-13T11:05:58+01:00
draft: false
---

### Prerequisites

* A working K8s cluster 
* Kubectl

see [this article](/articles/devops/deploying_kubernetes_clusters_with_different_cloud_providers)

### Creating a test back-end service

A `deployment` and `service` needs to be implemented to create :

* a pod with an application in this case called `marshyon/test-node-listener` - see [this repo](https://github.com/marshyon/test-node-listener) for the code behind this

* a service of type `ClusterIP `, the default if not specified and by its nature only available internally within the K8s cluster

> The latter is important to note as if this were to be of type `LoadBalancer`, the service would be exposed by the underlying cloud fabric and this would be a 1:1 relationship in that for every `service` of type `LoadBalancer` there would be a separate load balancer and external IP address created. With large or complicated applications this could quickly become unwieldy. By deploying an ingress controller, a ___single load balancer___ is created that can distribute requests to different back end services by host name and or urls.

Here is a manifest file for a deployment and service - `post.yaml` :

```
apiVersion: v1
kind: Service
metadata:
  name: post1
spec:
  ports:
  - port: 80
    targetPort: 4000
  selector:
    app: post1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: post1
spec:
  selector:
    matchLabels:
      app: post1
  replicas: 2
  template:
    metadata:
      labels:
        app: post1
    spec:
      containers:
      - name: post1
        image: marshyon/test-node-listener
        args:
        - "-text=post1"
        ports:
        - containerPort: 4000
```
it is applied with :

```
kubectl apply -f .\post1.yaml
```

the deployment can be seen to be running with :

```
kubectl.exe get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
post1   2/2     2            2           15m
```

running pods :

```
kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
post1-68fb864b7d-4clbh   1/1     Running   0          6m42s
post1-68fb864b7d-msz2x   1/1     Running   0          6m42s
```

### Setting Up Nginx Ingress Controller

Kubernetes has standard deployments for popular Cloud providers, at time of writing this includes Amazon, Azure, Google, Digital Ocean and Scaleway. For specific deployments see https://kubernetes.github.io/ingress-nginx/deploy

The following is for Digital Ocean :

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/do/deploy.yaml
```

Taking a look at the [url](https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/do/deploy.yaml) there are a number of manifests each separated by `---` character breaks and consisting of 10 kinds :

```
curl -s https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/do/deploy.yaml | egrep "^kind: " | cut -f2 -d' ' | grep -v Namespace | sort | uniq
ClusterRole
ClusterRoleBinding
ConfigMap
Deployment
Job
Role
RoleBinding
Service
ServiceAccount
ValidatingWebhookConfiguration
```

it is worth reviewing these as there are 17 in total :

```
curl -s https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/do/deploy.yaml | egrep "^kind: " | cut -f2 -d' ' | grep -v Namespace | sort | wc -l
17
```

and this can offer insight into how complex services are deployed in K8s and can also serve to inform how such ingress deployments may be crafted to suit applications for different applications and clients.

### Ingress configuration

To this point all of the infrastructure has been implemented but there is no exposure of the running service to the world outside of K8s.


The ingress configuration file : `ingress.yaml`

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: post-ingress
spec:
  rules:
  - host: post1.notapplicable.info
    http:
      paths:
      - backend:
          serviceName: post1
          servicePort: 80  
```

is applied with :

```
kubectl apply -f .\ingress.yaml
```

its status is obtained from :

```
kubectl get ingress
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME           CLASS    HOSTS                                                                        ADDRESS           PORTS   AGE
echo-ingress   <none>   post1.notapplicable.info   206.189.245.136   80      79m
```

Here the ip address `206.189.245.136` is created by the underlying `Digital Ocean` infrastructure.

A final step, entirely outside of K8s is required to create a `DNS A record` of `post1.notapplicable.info` ( replacing post1.notapplicable.info with the domain name that you own ).


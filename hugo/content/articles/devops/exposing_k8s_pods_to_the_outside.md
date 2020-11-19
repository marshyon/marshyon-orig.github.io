---
title: "Exposing Pods in K8s to the outside world"
date: 2020-11-07T14:23:58+01:00
authors: ["Jon Brookes"]
draft: false
---

### Simple single pod deploy and exposure of service outside of K8s

using a simple test nodejs docker container at :

https://hub.docker.com/r/marshyon/test-node-container

lets create a running pod using a pre-configured application from docker hub:

```
kubectl run node-test --image=marshyon/test-node-container --port=8080
```

this can be seen to be running within k8s by getting pods in the default namespace :

```
kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
node-test   1/1     Running   0          9m57s
```

to expose this to the outside world we need to expose it using a load balancer :

```
kubectl expose pod node-test --port=8080 --name=frontend --type=LoadBalancer
service/frontend exposed
```

to find out where it is now available we can list the clusters running services :

```
kubectl get service
NAME         TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)          AGE
frontend     LoadBalancer   10.3.241.8   35.189.125.205   8080:31713/TCP   71s
kubernetes   ClusterIP      10.3.240.1   <none>           443/TCP          25m
```

and to smoke test the service we can curl to it with :

```
curl http://35.189.125.205:8080
You've hit node-test
```

### Using a deployment to create a replica set and exposure outside of K8s with LoadBalancer

the following `deployment.yaml` file can be used to create a replica set of 'node-test-apps' that can be edited and redeployed.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-test-deployment
  labels:
    app: node-test-label
spec:
  replicas: 10
  selector:
    matchLabels:
      app: node-test-label
  template:
    metadata:
      labels:
        app: node-test-label
    spec:
      containers:
      - name: node-test-app
        image: marshyon/test-node-container
        ports:
        - containerPort: 8080
```

to apply this file :

```
kubectl apply -f deployment.yaml
```

this will create a replica set within a deployment - the identifier 'label' of `node-test-label` being used to link all of the 10 replicas

`kubectl get pods` will list 10 running pods

this deployment can now be exposed using a service of type LoadBalancer :

```
kubectl expose -f deployment.yaml --port=8080 --target-port=8080 --type=LoadBalancer
```

the service is created and can be seen with

```
kubectl get service
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
kubernetes             ClusterIP      10.3.240.1     <none>          443/TCP          2m58s
node-test-deployment   LoadBalancer   10.3.254.125   34.105.188.22   8080:32055/TCP   39s
```

and tested with 

```
curl http://34.105.188.22:8080
You've hit node-test-deployment-7745697756-thg6k
```

### Editing and re-applying a deployment

the above yaml file can be edited to have less or more replicas and re-applied with 

```
kubectl apply -f deployment.yaml
```

and the 10 replicas ( here edited to be 3 ) can be seen as :

```
kubectl get pods
NAME                                    READY   STATUS        RESTARTS   AGE
node-test-deployment-7745697756-7s8pv   1/1     Terminating   0          3m4s
node-test-deployment-7745697756-cxglf   1/1     Running       0          3m5s
node-test-deployment-7745697756-fq6f8   1/1     Terminating   0          3m4s
node-test-deployment-7745697756-h49mk   1/1     Terminating   0          3m4s
node-test-deployment-7745697756-p76z6   1/1     Running       0          3m4s
node-test-deployment-7745697756-pf6ch   1/1     Terminating   0          3m5s
node-test-deployment-7745697756-ptd7x   1/1     Terminating   0          3m5s
node-test-deployment-7745697756-rvfl2   1/1     Terminating   0          3m4s
node-test-deployment-7745697756-thg6k   1/1     Running       0          3m4s
node-test-deployment-7745697756-vfktd   1/1     Terminating   0          3m4s
```

after some time :

```
kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
node-test-deployment-7745697756-cxglf   1/1     Running   0          6m15s
node-test-deployment-7745697756-p76z6   1/1     Running   0          6m14s
node-test-deployment-7745697756-thg6k   1/1     Running   0          6m14s
```
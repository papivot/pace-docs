# Kubernetes fundamentals

## Kubectl fundamentals

#### Validate connectivity to cluster

Note: The generation and details of the kubeconfig file is addressed in a different section. 

To check is a valid kubeconfig file exists and kubectl get connect to the cluster using the config file ($HOME/.kube/config) - 
>`kubectl version`

If there are issues connecting with the K8s cluster, the output would be similar to this - 
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

else
it would be similar to this -
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:19:22Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```

#### Common kubectl output  formats

Some options on kubectl outputs - 

> `kubectl get nodes`

```shell
NAME                                   STATUS   ROLES    AGE   VERSION
045bbb78-a9cf-4087-b7d8-099f945f71ca   Ready    <none>   28d   v1.13.5
46be6e0b-127b-4338-9010-2cfe43e7cc38   Ready    <none>   28d   v1.13.5
...
```

>`kubectl get nodes -o wide`

```shell
NAME                                   STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
045bbb78-a9cf-4087-b7d8-099f945f71ca   Ready    <none>   28d   v1.13.5   172.28.0.5    172.28.0.5    Ubuntu 16.04.6 LTS   4.15.0-46-generic   docker://18.6.3
46be6e0b-127b-4338-9010-2cfe43e7cc38   Ready    <none>   28d   v1.13.5   172.28.0.6    172.28.0.6    Ubuntu 16.04.6 LTS   4.15.0-46-generic   docker://18.6.3
...
```

> `kubectl get nodes -o json`

```shell
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2019-05-14T20:15:44Z",
                "labels": {
...
```

#### Common kubectl commands

> `kubectl cluster-info`

```shell

```

> `kubectl get pods`

```shell
NAME                     READY   STATUS    RESTARTS   AGE
mysql-799956477c-8tm8x   1/1     Running   0          14d
```

> `kubectl get pods --all-namespaces`

```shell
NAMESPACE      NAME                                     READY   STATUS      RESTARTS   AGE
default        mysql-799956477c-8tm8x                   1/1     Running     0          14d
kube-ops       k8s-operations-5fcddbcf76-5b26c          1/1     Running     0          24d
kube-ops       k8s-operations-5fcddbcf76-hnjsx          1/1     Running     0          24d
kube-system    coredns-54586579f6-cpww8                 1/1     Running     0          28d
kube-system    coredns-54586579f6-f57b9                 1/1     Running     0          28d
...
```
>`kubectl get namespaces`

```shell
NAME            STATUS   AGE
default         Active   28d
kube-ops        Active   24d
kube-public     Active   28d
kube-system     Active   28d
...
```

> `kubectl describe pods [pod_name] -n [namespace]`

```shell
Name:               mysql-799956477c-8tm8x
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               d2560b7d-9522-4662-a342-dfbfe5946641/172.28.0.7
Start Time:         Mon, 27 May 2019 22:05:01 -0400
...
```

## Deploying your first pod

Note: We will leverage the Docker image that we created in the Docker fundamentals section. Since we uploaded the image in GCR - gcr.io/pa-nverma/k8soper:0.0.1 , we will reference GCR in this section for the image registry. If you are using a different registry, please modify the reference accordingly. 

> `kubectl run k8s-operations --image=gcr.io/pa-nverma/k8soper:0.0.1`

This should create a require deployment and pod artifacts in the default namespace 1ith an output similar to this -

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjMyNjA3MDA5LC0xNjgyNjQ0NzcxLC0xNj
U4ODU0MDk5LC0zMzU1Nzg1MzgsMTAxOTAwOTg5MSwxMTgzOTU0
Mjc2LDE2MzU5MTg4NzAsOTkxOTU1NDU3LDQ5NjYwNzM3OSwtMT
g2MzE0NzYxMyw3MzA5OTgxMTZdfQ==
-->
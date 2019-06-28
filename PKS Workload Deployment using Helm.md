
# PKS Workload Deployment using Helm

In this lab/demo, we will show you how to deploy a set of applications using Helm on a K8s cluster. This exercise serves multiple purposes - 

1. How to interact with Helm to deploy applications
2. Since the choice of application is Prometheus and Grafana, it will also show how to start with monitoring and reporting on a K8s cluster
3. Show how to interact with an application running on K8s cluster. 

### Requirements 

- A working K8s cluster with admin access.

## Deployment of Helm

To deploy the CLI, use brew on Mac

> `brew install kubernetes-helm`

Helm has a client server model where the server component - **tiller** -  runs on the Kubernetes cluster where helm targets the deployment. Helm uses the kubeconfig file to connect to the server component running on the K8s cluster. 

> `helm version`

without a valid kubeconfig you get this - 

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Error: Get http://localhost:8080/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp [::1]:8080: connect: connection refused
```

with a valid kubeconfig to the cluster and tiller installed you get something similar to this -

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwMTg5MDM5Myw5MzA4MDYwMTVdfQ==
-->
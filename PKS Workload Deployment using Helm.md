
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

Before installing the Tiller server component, we need to create a service account and give it some elevated privileges. Use the following yaml - 

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

> `kubectl apply -f rbac-config.yaml`

where rbac-config.yaml is the name of the file with the above contents. Once completed, perform the helm init to deploy the Tiller server component. 

> `helm init --service-account tiller`

This will install the server component. `kubectl get pods -n kube-system` should give something similar to this - 

```shell
NAME                                    READY   STATUS    RESTARTS   AGE
...
tiller-deploy-6d65d78679-p5cm9          1/1     Running   0          128m
...
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODM4MTQ2ODMsOTMwODA2MDE1XX0=
-->
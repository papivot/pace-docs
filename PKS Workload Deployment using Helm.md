
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
with a valid kubeconfig, where helm has not been initialized, you will get this -

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Error: could not find a ready tiller pod
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

where rbac-config.yaml is the name of the file with the above contents. Once completed, perform helm init to deploy the Tiller server component. 

> `helm init --service-account tiller`

This will install the server component. `kubectl get pods -n kube-system` should give something similar to this - 

```shell
NAME                                    READY   STATUS    RESTARTS   AGE
...
tiller-deploy-6d65d78679-p5cm9          1/1     Running   0          128m
...
```

> `helm version`

with a valid kubeconfig to the cluster and tiller installed you get something similar to this -

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
```

You are now ready to use helm to deploy applications in the cluster. 

## Deploy Prometheus

Before we start with the deployment we will create a namespace called **monitoring** and limit all work to the monitoring namespace.

> `kubectl create namespace monitoring`

To search for the Prometheus charts use the command - 

> `helm search prometheus`

should return something similar to this - 

```shell
NAME                                 	CHART VERSION	APP VERSION	DESCRIPTION
stable/prometheus                    	8.14.0       	2.11.1     	Prometheus is a monitoring system and time series database.
stable/prometheus-adapter            	1.2.0        	v0.5.0     	A Helm chart for k8s prometheus adapter
stable/prometheus-blackbox-exporter  	0.4.0        	0.14.0     	Prometheus Blackbox Exporter
...
stable/prometheus-nats-exporter      	2.1.0        	0.4.0      	A Helm chart for prometheus-nats-exporter
stable/prometheus-node-exporter      	1.5.1        	0.18.0     	A Helm chart for prometheus node-exporter
...
```

We will use the `stable/prometheus` chart for this session. 

Use the inspect command to get details on the chart - 

> `helm inspect stable/prometheus`

This should list a detailed output.

```shell
...
## TL;DR;

```console
$ helm install stable/prometheus

## Introduction

This chart bootstraps a [Prometheus](https://prometheus.io/) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.3+ with Beta APIs enabled

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm install --name my-release stable/prometheus
...
```

Use the following command to install prometheus and its required components - 

> `helm install stable/prometheus --namespace monitoring --name prometheus`

This should create the necessary artifacts and start the pods. Output will detail what was created as part of this install. 

```shell
NAME:   prometheus
LAST DEPLOYED: Fri Jun 28 15:01:40 2019
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
prometheus-alertmanager  1     1s
prometheus-server        3     1s

==> v1/PersistentVolumeClaim
NAME                     STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
prometheus-alertmanager  Pending  1s
prometheus-server        Pending  1s

==> v1/Pod(related)
NAME                                            READY  STATUS             RESTARTS  AGE
prometheus-alertmanager-668785dc56-d7bbm        0/2    Pending            0         1s
prometheus-kube-state-metrics-7bcf787fd4-lt2rf  0/1    ContainerCreating  0         1s
prometheus-node-exporter-9gfwp                  0/1    ContainerCreating  0         1s
prometheus-node-exporter-svxng                  0/1    ContainerCreating  0         1s
prometheus-node-exporter-sw744                  0/1    ContainerCreating  0         1s
prometheus-pushgateway-75dc77db76-4pjzj         0/1    ContainerCreating  0         1s
prometheus-server-5d5f6db7cc-mkpwg              0/2    Pending            0         1s

==> v1/Service
NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
prometheus-alertmanager        ClusterIP  10.100.200.147  <none>       80/TCP    1s
prometheus-kube-state-metrics  ClusterIP  None            <none>       80/TCP    1s
prometheus-node-exporter       ClusterIP  None            <none>       9100/TCP  1s
prometheus-pushgateway         ClusterIP  10.100.200.107  <none>       9091/TCP  1s
prometheus-server              ClusterIP  10.100.200.171  <none>       80/TCP    1s

==> v1/ServiceAccount
NAME                           SECRETS  AGE
prometheus-alertmanager        1        1s
prometheus-kube-state-metrics  1        1s
prometheus-node-exporter       1        1s
prometheus-pushgateway         1        1s
prometheus-server              1        1s

==> v1beta1/ClusterRole
NAME                           AGE
prometheus-kube-state-metrics  1s
prometheus-server              1s

==> v1beta1/ClusterRoleBinding
NAME                           AGE
prometheus-kube-state-metrics  1s
prometheus-server              1s

==> v1beta1/DaemonSet
NAME                      DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
prometheus-node-exporter  3        3        0      3           0          <none>         1s

==> v1beta1/Deployment
NAME                           READY  UP-TO-DATE  AVAILABLE  AGE
prometheus-alertmanager        0/1    1           0          1s
prometheus-kube-state-metrics  0/1    1           0          1s
prometheus-pushgateway         0/1    1           0          1s
prometheus-server              0/1    1           0          1s
...
```

Run the following to get the pod status - 

> `kubectl get pods -n monitoring`

should display something like this - 

```shell
NAME                                             READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-668785dc56-d7bbm         0/2     Pending   0          2m36s
prometheus-kube-state-metrics-7bcf787fd4-lt2rf   1/1     Running   0          2m36s
prometheus-node-exporter-9gfwp                   1/1     Running   0          2m36s
prometheus-node-exporter-svxng                   1/1     Running   0          2m36s
prometheus-node-exporter-sw744                   1/1     Running   0          2m36s
prometheus-pushgateway-75dc77db76-4pjzj          1/1     Running   0          2m36s
prometheus-server-5d5f6db7cc-mkpwg               0/2     Pending   0          2m36s
```

Notice that it may happen that some of the pods are in a pending state. To troubleshoot run the following - 

> `kubectl describe pod prometheus-server-5d5f6db7cc-mkpwg -n monitoring` 

where `prometheus-server-5d5f6db7cc-mkpwg` is the name of the pod from the previous output.  This should provide a status and a possible reason for failure - 

```shell
...
Events:
  Type     Reason            Age                   From               Message
  ----     ------            ----                  ----               -------
  Warning  FailedScheduling  24s (x10 over 4m37s)  default-scheduler  pod has unbound immediate PersistentVolumeClaims (repeated 3 times)
```

The pod is looking for some storage in the form of PersistantVolumeClaim and is unable to get it. We will have to provide it in the next few steps. 

Delete the prometheus release 

> `helm ls`

> `helm delete prometheus --purge`

#### Creating a storage class

If not already available, we will 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE5MjI1NDQ2OSw3NjExMzY0MzIsLTE4OD
M4MTQ2ODMsOTMwODA2MDE1XX0=
-->
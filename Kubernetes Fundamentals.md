# Kubernetes fundamentals

### Requirements
- Access to a working Kubernetes cluster (Pivotal employees connected thru the VPN can re
- Access to kubectl binary

## Kubectl fundamentals

#### Validate connectivity to cluster

**Note**: The generation and details of the kubeconfig file is addressed in a different section. 

To check if a valid kubeconfig file exists, and that kubectl gets connected to the cluster, use the config file ($HOME/.kube/config) - 
>`kubectl version`

If there are issues connecting with the K8s cluster, the output would be similar to this - 
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```


or it would be similar to this -
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:19:22Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```

#### Common kubectl output  formats

Some kubectl output formats - 

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
Kubernetes master is running at https://cluster00.gcpcloud.navneetv.com:8443
Heapster is running at https://cluster00.gcpcloud.navneetv.com:8443/api/v1/namespaces/kube-system/services/heapster/proxy
CoreDNS is running at https://cluster00.gcpcloud.navneetv.com:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
...
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

**Note**: We will leverage the Docker image that we created in the Docker fundamentals section. Since we uploaded the image in GCR - gcr.io/pa-nverma/k8soper:0.0.1 , we will reference GCR in this section for the image registry. If you are using a different registry, please modify the reference accordingly. 

#### Step 1 - Run the Docker image from the registry

> `kubectl run k8s-operations --image=gcr.io/pa-nverma/k8soper:0.0.1`

This should create a require deployment and pod artifacts in the default namespace with an output similar to this -

```shell
deployment.apps/k8s-operations created
```

#### Step 2 - View the running configuration 

Wait for a minute or so for the pod to be successfully scheduled and then execute kubectl to view the pod - 

> `kubectl get pods -n default -o wide`

```shell
NAME                              READY   STATUS    RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GATES
k8s-operations-6f97c49687-rbcqv   1/1     Running   2          8m18s   10.200.56.15   vm-ea04c1fe-045a-466e-7c2f-3b7ce8d5f4c2   <none>           <none>
```
Use the node name value to execute the next command - 

> `kubectl get pods k8s-operations-6f97c49687-rbcqv -n default -o yaml`

This should output a yaml configuration of your deployment - 

```shell
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2019-06-17T14:33:12Z"
  generateName: k8s-operations-6f97c49687-
  labels:
    pod-template-hash: 6f97c49687
    run: k8s-operations
  name: k8s-operations-6f97c49687-rbcqv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: k8s-operations-6f97c49687
...
...
```

Congratulations! You executed your first pod in a Kubernetes cluster. 

#### Step 3 - View the POD logs

Using the name of the POD retrieved in Step 2, run the following command - 

> `kubectl logs k8s-operations-6f97c49687-rbcqv -n default`

You may get an output similar to this - 

```shell
job started at:  2019-06-17 14:49:56.693852
Traceback (most recent call last):
  File "/usr/local/bin/exportjson.py", line 115, in <module>
    schedule.run_pending()
  File "/usr/lib/python3.6/site-packages/schedule/__init__.py", line 563, in run_pending
    default_scheduler.run_pending()
  File "/usr/lib/python3.6/site-packages/schedule/__init__.py", line 94, in run_pending
    self._run_job(job)
  File "/usr/lib/python3.6/site-packages/schedule/__init__.py", line 147, in _run_job
    ret = job.run()
  File "/usr/lib/python3.6/site-packages/schedule/__init__.py", line 466, in run
    ret = self.job_func()
  File "/usr/local/bin/exportjson.py", line 15, in job
    clustername = os.environ['CLUSTER_NAME']
  File "/usr/lib/python3.6/os.py", line 669, in __getitem__
    raise KeyError(key) from None
KeyError: 'CLUSTER_NAME'
```
**Note**:  This outputs the stderr/stdout stream of the pod and displays some of the errors that the container generated during the execution. 

#### Step 4 - Running commands in your container

Using the name of the POD retrieved in step2, run the following command - 

>`kubectl exec -it k8s-operations-6f97c49687-rbcqv -- /bin/sh`

This should return a shell prompt in the container - 

```shell
~ $
```
executing ls -la / lists the root file system within the container - 

> `ls -al /`

```shell
~ $ ls -la /
total 72
drwxr-xr-x    1 root     root          4096 Jun 17 14:57 .
drwxr-xr-x    1 root     root          4096 Jun 17 14:57 ..
-rwxr-xr-x    1 root     root             0 Jun 17 14:57 .dockerenv
drwxr-xr-x    2 root     root          4096 May  9 20:49 bin
drwxr-xr-x    5 root     root           360 Jun 17 14:57 dev
drwxr-xr-x    1 root     root          4096 Jun 17 14:57 etc
drwxr-xr-x    2 root     root          4096 May  9 20:49 home
drwxr-xr-x    1 root     root          4096 May  9 20:49 lib
drwxr-xr-x    5 root     root          4096 May  9 20:49 media
drwxr-xr-x    2 root     root          4096 May  9 20:49 mnt
drwxr-xr-x    2 root     root          4096 May  9 20:49 opt
```
> `exit`

to exit the shell prompt. 

#### Step 5 - Cleanup

Using the name of the POD retrieved in Step 2, run the following command -

> `kubectl delete pods k8s-operations-6f97c49687-ghmwf -n default`

This should delete the running pod with an output similar to this - 
```shell
pod "k8s-operations-6f97c49687-ghmwf" deleted
```

`Kubectl get pods` will show that another instance of the pod has been created. This is due to the fact that there is a higher level K8s parent object called deployment that is the controlling the child POD. To perform a complete cleanup, the top level object has to be destroyed to prevent the POD from spawning once again. 

To do so, execute the following command - 

> `kubectl delete deployment k8s-operations -n default`

```shell
deployment.extensions "k8s-operations" deleted
```

> `kubectl get pods -n default`

The above command should not return any k8s-operations-* pods.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc3MjU4NzU2OSwtNjk5NTk5Mjk5LDMzMD
YwODEwLC0yMDU4OTM1NTg4XX0=
-->
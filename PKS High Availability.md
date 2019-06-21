
# PKS High Availability

PKS provides high availability at 4 different layers -

 1. HA of K8s PODs
 2. HA of K8s nodes
 3. HA of K8s services
 4. HA across availability zones

## HA of K8s PODs

This is a function natively provided by Kubernetes. More details on pods lifecycle can be found [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/). If an involuntary disruption of the pods happen, K8s and its components try to restore the state of the application (thereby the pods) based on certain rules. The controller is responsible for the restore/recovery of the pods. 

- Set up a session to watch the nodes and their status 
> ` watch -d kubectl get pods -o wide`

```shell
Every 2.0s: kubectl get pods -o wide -n pks-system                                                                                 Navneets-MBP.navlab.io: Fri Jun 21 16:15:27 2019

NAME                                    READY   STATUS      RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GATES
cert-generator-v0.19-gnqvk              0/1     Completed   0          4d      10.200.95.5    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
event-controller-5c764cbc6-fw74x        2/2     Running     489        4d      10.200.67.4    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
fluent-bit-2r4d8                        3/3     Running     0          2d22h   10.200.95.8    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
fluent-bit-nxz94                        3/3     Running     0          2d22h   10.200.67.10   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
fluent-bit-qcgj8                        3/3     Running     0          54m     10.200.3.2     vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   <none>           <none>
...
```



## HA of K8s nodes

- Make sure that the `Enable VM Resurrector Plugin` setting is checked in the BOSH director tile. If not, enable it and apply changes. 

- Set up a session to watch the nodes and their status 
> ` watch -d kubectl get nodes -o wide`

- GO to your IaaS provider console and delete/destroy one of the worker nodes.  
- Within a few minutes, the impacted node goes from Ready to NonReady to missing from the kubectl output. 

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:05 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-0d33fa8f-9acc-444c-50b8-afec139f933f   Ready    <none>   3d23h   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:22 2019

NAME                                      STATUS     ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-0d33fa8f-9acc-444c-50b8-afec139f933f   NotReady   <none>   3d23h   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready      <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready      <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3

```

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:40 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3

```

- Run this command to watch the pods placement. 
> ` watch -d kubectl get pods -o wide --all-namespaces`

- You will notice that all PODs have been restarted on other nodes (except for PODs that are part of daemonsets). 

```shell
Every 2.0s: kubectl get pods -o wide --all-namespaces                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:23:59 2019

NAMESPACE     NAME                                    READY   STATUS      RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GA
TES
kube-system   coredns-54586579f6-gdhtd                1/1     Running     4          3d23h   10.200.67.2    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
kube-system   coredns-54586579f6-rz72s                1/1     Running     4          3d23h   10.200.95.2    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
kube-system   coredns-54586579f6-t9dwp                1/1     Running     0          2m8s    10.200.67.15   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
...
```

- Within a few minutes the kubectl get nodes window will show a new node has joined the cluster and is taking workloads -

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 15:24:11 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   Ready    <none>   3m49s   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   4d      v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   4d      v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```

## HA of K8s services

- Set up a session to watch the nodes and their status 
> ` watch -d kubectl get nodes -o wide`

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 15:39:02 2019

NAME                                      STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   Ready    <none>   18m   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   4d    v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   4d    v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```
- Login to one of the worker VMs. 
- We will kill the kubelet  process on the worker VM. 
> `ps -eaf|grep kubelet`

```shell
root     12766     1  0 Jun17 ?        00:00:00 /bin/bash -ex /var/vcap/jobs/kubelet/bin/kubelet_ctl start
root     12813 12766  3 Jun17 ?        02:56:31 kubelet --cni-bin-dir=/var/vcap/jobs/kubelet/packages/cni/bin --container-runtime=docker --docker=unix:///var/vcap/sys/run/docker/docker.sock --docker-endpoint=unix:///var/vcap/sys/run/docker/docker.sock --kubeconfig=/var/vcap/jobs/kubelet/config/kubeconfig --network-plugin=cni --root-dir=/var/vcap/data/kubelet --cloud-config=/var/vcap/jobs/kubelet/config/cloud-provider.ini --cloud-provider=gce --hostname-override=vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97 --node-labels=pks-system/cluster.name=gcpcluster00,pks-system/cluster.uuid=service-instance_47267983-9989-41ce-bc33-905e249b1fbc,spec.ip=10.0.11.6,bosh.id=29e4a450-fbfc-404e-aa92-772f678e1693,bosh.zone=us-east1-b --config=/var/vcap/jobs/kubelet/config/kubeletconfig.yml
```
- Grab the PID for the kubelet process. 12813 in this example.
- Kill the kubelet process.
>`sudo kill -9 12813`

- You will notice that the Node momentarily becomes `NotReady` and then changes back to ready. 

```shell
very 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 15:39:52 2019

NAME                                      STATUS     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   Ready      <none>   19m   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   NotReady   <none>   4d    v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready      <none>   4d    v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```

- Check the process table on the Worker VM. The kubelet process is back up. 
> `ps -eaf|grep kubelet`

```shell
root     24933     1  0 19:39 ?        00:00:00 /bin/bash -ex /var/vcap/jobs/kubelet/bin/kubelet_ctl start
root     24981 24933  3 19:39 ?        00:00:23 kubelet --cni-bin-dir=/var/vcap/jobs/kubelet/packages/cni/bin --container-runtime=docker --docker=unix:///var/vcap/sys/run/docker/docker.sock --docker-endpoint=unix:///var/vcap/sys/run/docker/docker.sock --kubeconfig=/var/vcap/jobs/kubelet/config/kubeconfig --network-plugin=cni --root-dir=/var/vcap/data/kubelet --cloud-config=/var/vcap/jobs/kubelet/config/cloud-provider.ini --cloud-provider=gce --hostname-override=vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97 --node-labels=pks-system/cluster.name=gcpcluster00,pks-system/cluster.uuid=service-instance_47267983-9989-41ce-bc33-905e249b1fbc,spec.ip=10.0.11.6,bosh.id=29e4a450-fbfc-404e-aa92-772f678e1693,bosh.zone=us-east1-b --config=/var/vcap/jobs/kubelet/config/kubeletconfig.yml
```

## HA across AZs

While this HA is difficult to demonstrate (by performing an AZ level failure), it can be demonstrated by the following - 
 
- Verify that within the PKS tile, `## Assign AZs and Networks` -> `Balance other jobs in` has multiple AZs selected. This will make sure that all non-singleton VM types get deployed in multiple AZs. 
- If the cluster has multiple masters, verify that they are placed across different AZs.
- If the cluster has multiple worker nodes, verify that they are placed across different AZs. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjI1NzI0MzUwLC0xNDYwODY0ODY3LDQ4MD
M5MDc5Myw3NTI5MjI2OTksOTU2Njc2OTc4LC0xMzQ2NjEzMDU2
LC05OTgxMzk1NzAsLTUwMTM3NjE3LC0xNTU4MjcxMDk3LC0xNj
I1ODgwMTk2LDczMDk5ODExNl19
-->
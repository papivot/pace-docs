
# PKS High Availability

PKS provides high availability at 4 different layers -

 1. HA of K8s PODs
 2. HA of K8s nodes
 3. HA of K8s services
 4. HA across availability zones

## HA of K8s PODs

This is a function natively provided by Kubernetes. More details on pods lifecycle can be found [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/). If an involuntary disruption of the pods happen, K8s and its components try to restore the state of the application (thereby the pods) based on certain rules. The controller is responsible for the restore/recovery of the pods. 

## HA of K8s nodes

Make sure that the `Enable VM Resurrector Plugin` setting is checked in the BOSH director tile. If not, enable it and apply changes. 

Set up a session to watch the nodes and their status 
> ` watch -d kubectl get nodes -o wide`

GO to your IaaS provider console and delete/destroy one of the worker nodes.  Within a few minutes, the impacted node goes from Ready to NonReady to missing from the kubectl output. 

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

Run this command to watch the pods placement. 
> ` watch -d kubectl get pods -o wide --all-namespaces`

You will notice that all PODs have been restarted on other nodes (except for PODs that are part of daemonsets). 

```shell
Every 2.0s: kubectl get pods -o wide --all-namespaces                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:23:59 2019

NAMESPACE     NAME                                    READY   STATUS      RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GA
TES
kube-system   coredns-54586579f6-gdhtd                1/1     Running     4          3d23h   10.200.67.2    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
kube-system   coredns-54586579f6-rz72s                1/1     Running     4          3d23h   10.200.95.2    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
kube-system   coredns-54586579f6-t9dwp                1/1     Running     0          2m8s    10.200.67.15   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
...

```

Within a few minutes the kubectl get nodes window will show a new node has joined the cluster and is taking wor -

```shell
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 15:24:11 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   Ready    <none>   3m49s   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   4d      v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   4d      v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```

## HA of K8s services

## HA across AZs


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY4Mjg5OTM1NCwtMTM0NjYxMzA1NiwtOT
k4MTM5NTcwLC01MDEzNzYxNywtMTU1ODI3MTA5NywtMTYyNTg4
MDE5Niw3MzA5OTgxMTZdfQ==
-->
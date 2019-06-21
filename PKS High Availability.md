
# PKS High Availability

PKS provides high availability at 4 different layers -

 1. HA of K8s PODs
 2. HA of K8s nodes
 3. HA of K8s services
 4. HA across availability zones

## HA of K8s PODs

This is a function natively provided by Kubernetes. More details on pods lifecycle can be found [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/). If an involuntary disruption of the pods happen, K8s and its components try to restore the state of the application (thereby the pods) based on certain rules. The controller is responsible for the restore/recovery of the pods. 

## HA of K8s nodes
> ` watch -d kubectl get nodes -o wide`

```
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:05 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-0d33fa8f-9acc-444c-50b8-afec139f933f   Ready    <none>   3d23h   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
```

```
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:22 2019

NAME                                      STATUS     ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-0d33fa8f-9acc-444c-50b8-afec139f933f   NotReady   <none>   3d23h   v1.13.5   10.0.11.8                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready      <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready      <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3

```

```
Every 2.0s: kubectl get nodes -o wide                                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:21:40 2019

NAME                                      STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   3d23h   v1.13.5   10.0.11.6                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   3d23h   v1.13.5   10.0.11.7                   Ubuntu 16.04.6 LTS   4.15.0-50-generic   docker://18.6.3

```

## HA of K8s services

## HA across AZs


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQwMDE0NDkwMiwtOTk4MTM5NTcwLC01MD
EzNzYxNywtMTU1ODI3MTA5NywtMTYyNTg4MDE5Niw3MzA5OTgx
MTZdfQ==
-->
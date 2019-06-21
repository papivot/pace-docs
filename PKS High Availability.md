
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

GO to your IaaS provider console and delete/destroy one of the worker nodes.  Within a few minutes, the impa
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

> ` watch -d kubectl get pods -o wide --all-namespaces`

```shell
Every 2.0s: kubectl get pods -o wide --all-namespaces                                                                              Navneets-MBP.navlab.io: Fri Jun 21 14:23:59 2019

NAMESPACE     NAME                                    READY   STATUS      RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GA
TES
kube-system   coredns-54586579f6-gdhtd                1/1     Running     4          3d23h   10.200.67.2    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
kube-system   coredns-54586579f6-rz72s                1/1     Running     4          3d23h   10.200.95.2    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
kube-system   coredns-54586579f6-t9dwp                1/1     Running     0          2m8s    10.200.67.15   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
kube-system   kubernetes-dashboard-6c68548bc9-4rscr   1/1     Running     0          3d23h   10.200.67.3    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
kube-system   metrics-server-5475446b7f-xhl2w         1/1     Running     0          3d23h   10.200.95.3    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    cert-generator-v0.19-gnqvk              0/1     Completed   0          3d23h   10.200.95.5    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    event-controller-5c764cbc6-fw74x        2/2     Running     482        3d23h   10.200.67.4    vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    fluent-bit-2r4d8                        3/3     Running     0          2d20h   10.200.95.8    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    fluent-bit-nxz94                        3/3     Running     0          2d20h   10.200.67.10   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    kube-state-metrics-99659db4f-t55j8      2/2     Running     0          2m8s    10.200.67.17   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    metric-controller-585878fc8c-xrgf8      1/1     Running     0          2m8s    10.200.67.16   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    observability-manager-8d95f455d-rn4ld   1/1     Running     0          2m8s    10.200.67.13   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    sink-controller-b6bbd7d68-98j5b         1/1     Running     5          3d23h   10.200.95.4    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    telegraf-d4jqf                          1/1     Running     0          3d23h   10.0.11.7      vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    telegraf-dp89x                          1/1     Running     0          3d23h   10.0.11.6      vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    telemetry-agent-776d45f8d8-sqn5t        1/1     Running     0          2d20h   10.200.67.11   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    validator-8568fd5c8f-vqspt              1/1     Running     0          2m5s    10.200.95.9    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>
pks-system    wavefront-collector-dbdf7b867-k5rnf     1/1     Running     0          2m8s    10.200.67.14   vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   <none>           <none>
pks-system    wavefront-proxy-f79894746-mc8v9         1/1     Running     0          3d23h   10.200.95.7    vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   <none>           <none>

```

Within a few minutes -

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
eyJoaXN0b3J5IjpbLTExNDEyOTYwNTIsLTEzNDY2MTMwNTYsLT
k5ODEzOTU3MCwtNTAxMzc2MTcsLTE1NTgyNzEwOTcsLTE2MjU4
ODAxOTYsNzMwOTk4MTE2XX0=
-->
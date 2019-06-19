
# PKS High Availability

PKS provides high availability at 4 different layers -

 1. HA of K8s PODs
 2. HA of K8s nodes
 3. HA of K8s services
 4. HA across availability zones

## HA of K8s PODs

This is a function natively provided by Kubernetes. More details on pods lifecycle can be found [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/). If an involuntary disruption of the pods happen, K8s and its components try to restore the state of the application (thereby the pods) based on certain rules. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzOTE4MTA0NTAsLTE1NTgyNzEwOTcsLT
E2MjU4ODAxOTYsNzMwOTk4MTE2XX0=
-->
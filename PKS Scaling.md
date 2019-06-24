
## Node Scaling in PKS

Using the PKS CLI, login to the PKS API endpoint

> `pks login -k -a api.pks.domain.com -u userid`                                                                       

Provide the password and login

```shell
Password: ********
API Endpoint: api.pks.domain.com
User: userid
```
Get the details of the cluster that needs to be scaled out, for example - 

> `pks cluster gcpcluster00 `                                                                                                        

should return 
```shell
Name:                     gcpcluster00
Plan Name:                small
UUID:                     47267983-9989-41ce-bc33-905e249b1fbc
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance provisioning completed
Kubernetes Master Host:   cluster00.domain.com
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  10.0.11.5
Network Profile Name:
```
Validate the number of nodes on the cluster using kubectl - 

> `kubectl get nodes`

should return 3 nodes - 
```shell                                                                                           
NAME                                      STATUS   ROLES    AGE     VERSION
vm-1391f084-4af3-47c6-613d-03e5f3b8abb3   Ready    <none>   2d20h   v1.13.5
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   6d21h   v1.13.5
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   6d21h   v1.13.5
```

Use the PKS CLI to scale out the cluster from 3 worker nodes to 4 worker nodes - 

> `pks update-cluster gcpcluster00 --num-nodes 4 `

Confirm to validate - 
```shell
Update summary for cluster gcpcluster00:
Worker Number: 4
Are you sure you want to continue? (y/n): y
Use 'pks cluster gcpcluster00' to monitor the state of your cluster
```

Check the status until an additional node has been successfully added to the cluster - 

> `pks cluster gcpcluster00 `                                                                                                        

```shell
Name:                     gcpcluster00
Plan Name:                small
UUID:                     47267983-9989-41ce-bc33-905e249b1fbc
Last Action:              UPDATE
Last Action State:        in progress
Last Action Description:  Instance update in progress
...
```
Within a few mins, the status should change to succeeded - 

```shell
Name:                     gcpcluster00
Plan Name:                small
UUID:                     47267983-9989-41ce-bc33-905e249b1fbc
Last Action:              UPDATE
Last Action State:        succeeded
...
Worker Nodes:             4
...
```
kubectl should also report 4 nodes - 

>`kubectl get nodes`

```shell
NAME                                      STATUS   ROLES    AGE     VERSION
vm-3c19c919-4a4f-4d59-7cfa-b6e3de2bfadd   Ready    <none>   15m     v1.13.5
vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97   Ready    <none>   6d21h   v1.13.5
vm-d3a0741d-bc56-4767-4434-95f61922ef2e   Ready    <none>   9m24s   v1.13.5
vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784   Ready    <none>   6d21h   v1.13.5
```
**Note** Using the options provided in PKS CLI, you can build automation around the node scaling function. PKS CLI can be integrated with performance tools to scale up (and down) based on node usage.  

**Cleanup** - Reduce the cluster node count back to 3.

>`pks update-cluster gcpcluster00 --num-nodes 3`

## Pod Scaling in Kubernetes

The demo provided here based on the autoscaling example provided [here](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

Make sure you have Kubectl admin access to the K8s cluster. 

Run the below command to start a modified php-apache deployment using the hpa-example image.

> ```
> kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
> ```

Should give an output similar to this - 

```shell
service/php-apache created
deployment.apps/php-apache created
```

Check the pod deployed as part of this deployment - 

> `kubectl get pods`

Should output one pod running. 

```shell
NAME                          READY   STATUS    RESTARTS   AGE
php-apache-84cc7f889b-cf62m   1/1     Running   0          16m
```
Create a Horizontal Pod Autoscaler (HPA) - 

> ```
>kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
>``` 

Upon successful completion, it should output - 

```
horizontalpodautoscaler.autoscaling/php-apache autoscaled
```

Get the hpa status - 

> `kubectl get hpa` 

Should display the following output - 

```shell
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          69s
```

Now, launch a busybox pod that will be used to connect to the php-apache pod (exposed thru the service) and generate load. 

> `kubectl run -i --tty load-generator --image=busybox /bin/sh`

Should give an output -

```shell
If you don't see a command prompt, try pressing enter.
/ #
```

In a second kubectl window check the pods running - 

> `kubectl get pods`

```shell 
NAME                              READY   STATUS    RESTARTS   AGE
load-generator-557649ddcd-b6fhh   1/1     Running   0          8m37s
php-apache-84cc7f889b-cf62m       1/1     Running   0          25m
```

In the original window, execute the following command in the load-generator pod's command prompt - 

> `while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done`

This should start generating the load and you will see output similar to this -

```
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!O
...
```
In the second window check the hpa -

> `kubectl get hpa`

Within a few mins, the load generated
kubectl get hpa                                                                                                                  
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   480%/50%   1         10        8          25m

kubectl get pods                                                                                                                 
NAME                              READY   STATUS    RESTARTS   AGE
load-generator-557649ddcd-b6fhh   1/1     Running   0          12m
php-apache-84cc7f889b-2srm8       1/1     Running   0          37s
php-apache-84cc7f889b-6cnq8       1/1     Running   0          52s
php-apache-84cc7f889b-7xzzc       1/1     Running   0          52s
php-apache-84cc7f889b-8zhmb       1/1     Running   0          52s
php-apache-84cc7f889b-9pmvc       1/1     Running   0          37s
php-apache-84cc7f889b-cf62m       1/1     Running   0          29m
php-apache-84cc7f889b-g9kk8       1/1     Running   0          67s
php-apache-84cc7f889b-hfqk7       1/1     Running   0          52s
php-apache-84cc7f889b-jzbxq       1/1     Running   0          68s
php-apache-84cc7f889b-rwbsg       1/1     Running   0          67s

kubectl get hpa                                                                                                                  
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   46%/50%   1         10        10         26m
<!--stackedit_data:
eyJoaXN0b3J5IjpbODQxMTI1MDc1LDE2OTIzNTM4MTEsMTY0OT
YyMzYwOSwtMTY3NzIzMzk3NywtMTUwMzk3ODgyNiw5MzEzMjIx
NTYsMTgzMTA5MjU3LDE5NjUyOTYxMTQsNzMwOTk4MTE2XX0=
-->
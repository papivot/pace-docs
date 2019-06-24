
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
**Note** Currently PKS does not allow 

## Pod Scaling in Kubernetes
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk1NTcxODE1NywtMTY3NzIzMzk3NywtMT
UwMzk3ODgyNiw5MzEzMjIxNTYsMTgzMTA5MjU3LDE5NjUyOTYx
MTQsNzMwOTk4MTE2XX0=
-->
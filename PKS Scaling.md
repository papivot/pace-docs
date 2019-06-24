
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

Use the PKS CLI to scale out the cluster from 3 worker nodes to 4 worker nodes - 

> ` `



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NzEwNTgyMjYsMTgzMTA5MjU3LDE5Nj
UyOTYxMTQsNzMwOTk4MTE2XX0=
-->
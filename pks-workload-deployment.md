
# PKS Workload Deployment using Helm

In this lab/demo, we will show you how to deploy a set of applications using yaml configuration files on a K8s cluster. This exercise serves multiple purposes - 

1. How to interact with yaml files to deploy applications
2. Show how to interact with an application running on K8s cluster. 

### Requirements 
- A working K8s cluster with admin access. (*Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Upon a PKS environment has been created, you can login using the pks cli and create a cluster using an active plan.)*

### Preparation

Login to the PKS API endpoint

>`pks login -k -a api.pks.domain.com -u username`                                                                                                 

```shell
Password: ********
API Endpoint: api.pks.domain.com
User: username
```

Get the list of clusters deployed - 

> `pks clusters`

should output the clusters deployed 

```shell
Name          Plan Name  UUID                                  Status     Action
gcpcluster00  small      69c4444c-8bf9-4907-956e-d94db7e145c2  succeeded  CREATE
```

>` pks get-credentials cluster_name`

will generate the kubeconfig file for the admin user `username` and allow kubectl commands to be executed against this cluster.

```shell
Fetching credentials for cluster gcpcluster00.
Context set for cluster gcpcluster00.

You can now switch between clusters by using:
$kubectl config use-context <cluster-name>
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI0OTUxNDQwMywtMTExODI0OTUzNF19
-->
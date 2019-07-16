
# PKS Bootcamp prep and pre-requisites 

The following artifacts needs to be readily available before the start of the bootcamp -

 - [ ] An account with [Papertrail](https://papertrailapp.com/) for logging
 - [ ] An account with [Jumpcloud](https://jumpcloud.com/signup/) for LDAP integration
 - [ ] Ops Manager with PKS installed and configured. The recommended option is GCP, as it is easy to deploy, manage, and the least expensive of the public clouds. Pivotal employees connected through the VPN can request a PKS environment in GCP through [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Toolsmith has a help video on how to request a PKS environment on GCP. 
 - [ ] BOSH Director should have the `Enable VM Resurrector Plugin` already enabled.
 - [ ] Jumpbox/bastion host with access to the PKS environment (preferably Ubuntu Linux) with the following binaries installed -
 * Kubectl - After signing into the Pivotal network, the relevant binaries can be downloaded from [here](https://network.pivotal.io/products/pivotal-container-service/#/releases/386533/file_groups/1831)
 * pks - After signing into the Pivotal network, the relevant binaries can be downloaded from [here](https://network.pivotal.io/products/pivotal-container-service/#/releases/386533/file_groups/1830).
 * uaac - this is generally installed as a Ruby gem 
> `gem install cf-uaac`
 * Curl
 * jq
 * om - The required binary can be deployed [here](https://github.com/pivotal-cf/om/releases)
 
 
 - [ ] PKS tile deployed with at least one plan that has the ability to deploy K8s clusters with `Allow privilaged` setting enabled.
 - [ ] PKS tile with `UAA as OIDC provider` **unchecked**
 - [ ] A K8s cluster deployed in the PKS environment. 
	 - [ To deploy a K8s cluster, use the following steps - 
 
>``
$ UAA_PWD=`om -t [opsman_fqdn] -u [opsman_userid] -p [opsman_password] -k credentials --product-name pivotal-container-service --credential-reference .properties.pks_uaa_management_admin_client -t json|jq -r .secret`
``

> `$ echo $UAA_PWD`

should output a non-empty string for e.g.
`MgHXgeuYBaTddbAuY1bklOmf3PV-nCZ-`

Use the UAAC cli to connect to the PKS API UAA endpoint

> `uaac target https://[api.pks.fqdn]:8443 --skip-ssl-validation`

should return 
```bash
Unknown key: Max-Age = 86400

Target: https://[api.pks.fqdn]:8443
```
Authenticate using the UAAC Cli

> `uaac token client get admin -s $UAA_PWD`

should return -
```bash
Unknown key: Max-Age = 86400

Successfully fetched token via client credentials grant.
Target: https://[api.pks.fqdn]:8443
Context: admin, from client admin
```
Use the UAAC cli to add a user -

>`$ uaac user add [username] --emails [username@email.com] -p [password]`

```bash
user account successfully added
```
Add the user to the **pks.cluster.admin** group -
> `uaac member add pks.clusters.admin [username]`

```
success
```
Use the PKS CLI to login to the PKS endpoint using the newly created user -

> `pks login -a [api.pks.fqdn] -u [username] -k`

```bash
Password: ********
API Endpoint: api.pks.fqdn
User: username
```

Use the PKS CLI to view the available plans - 

>`$ pks plans`

should return something similar to this - 
```shell
Name    ID                                    Description
small   8A0E21A8-8072-4D80-B365-D1F502085560  Example: This plan will configure a lightweight kubernetes cluster. Not recommended for production workloads.
medium  58375a45-17f7-4291-acf1-455bfdc8e371  Example: This plan will configure a medium sized kubernetes cluster, suitable for more pods.
large   241118e5-69b2-4ef9-b47f-4d2ab071aff5  Example: This plan will configure a large kubernetes cluster for resource heavy workloads, or a high number of workloads.
```

Use the PKS CLI to create a cluster using one of the above plans - 

>`pks create-cluster gcpcluster00 --external-hostname [clustername.fqdn]  --plan small`

should return 
```shell
Name:                     gcpcluster00
Plan Name:                small
UUID:                     69c4444c-8bf9-4907-956e-d94db7e145c2
Last Action:              CREATE
Last Action State:        in progress
Last Action Description:  Creating cluster
Kubernetes Master Host:   [clustername.fqdn]
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  In Progress
Network Profile Name:

Use 'pks cluster gcpcluster00' to monitor the state of your cluster
```

After a few minutes, the deployment would have successfully completed and 

>` pks cluster gcpcluster00`

should return something similar

```bash
Name:                     gcpcluster00
Plan Name:                small
UUID:                     69c4444c-8bf9-4907-956e-d94db7e145c2
Last Action:              CREATE
Last Action State:        succeeded
Last Action Description:  Instance provisioning completed
Kubernetes Master Host:   [clustername.fqdn]
Kubernetes Master Port:   8443
Worker Nodes:             3
Kubernetes Master IP(s):  10.0.11.10
Network Profile Name:
```

If this is a Toolsmith deployed GCP cluster, get the instance ID(s) of the master node(s) - they will have a label of `job: master`, and add these instance(s) to the backend configuration of the *-pks-cluster-1 loadbalancer.

To validate the cluster is in a working condition, execute the following command -

> `pks get-credentials gcpcluster00`

should return 

```shell
Fetching credentials for cluster gcpcluster00.
Context set for cluster gcpcluster00.

You can now switch between clusters by using:
$kubectl config use-context <cluster-name>
```

Get the nodes status of the cluster - 

> `kubectl get nodes`

should return something similar to this - 

```shell
NAME                                      STATUS   ROLES    AGE     VERSION
vm-264981be-c5da-45cc-6fc1-f8e9f10108f3   Ready    <none>   4h55m   v1.13.5
vm-739344f0-516a-4e34-4527-9b3cd9b553fc   Ready    <none>   4h52m   v1.13.5
vm-f918f979-757b-4214-5921-9a0a91bbe5df   Ready    <none>   70m     v1.13.5
```

 - [ ] Working Harbor registry. If you do not have one, use the following steps to deploy a Harbor tile along with the PKS tile. Details of the configuration and howto is provided [here](https://docs.pivotal.io/partners/vmware-harbor/installing.html).

- Download the Harbor product tile from Pivnet
- Login to OpsManager and upload the Harbor product tile using the `Import a Product` function. 
- Validate that the valid stemcell is associated with the tile.
- Configure the tile. Use the defaults (wherever possible) .
- Make sure Clair and Notary are enabled. 
-  Clair Updater interval (Hours) is set to a number greater than 0
- Apply changes. 
- Once configuration is completed, make sure that the DNS is updated to reflect the public IP and FQDN. Adjust the firewall, if necessary. 
- Validate the FQDN is reachable. 




<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMzMDA0NzAwNSwtMTA2MTc1NTg0M119
-->
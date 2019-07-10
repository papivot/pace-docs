
The following artifacts needs to be available before the start of the bootcamp -

 - Ops Manager with PKS installed and configured. The recommended option is GCP as it is easy to deploy, manage and the least expensive of the public clouds. Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Toolsmith has a help video on how to request a PKS environment on GCP. 
 - BOSH Director should have the `Enable VM Resurrector Plugin` already enabled.
 - Jumpbox/bastion host with access to the PKS environment (preferably Ubuntu Linux) with the following binaries installed -
 - [ ] Kubectl - After signing into the Pivotal network, the relevent binaries can be downloaded from [here](https://network.pivotal.io/products/pivotal-container-service/#/releases/386533/file_groups/1831)
 - [ ] pks - After signing into the Pivotal network, the relevent binaries can be downloaded from [here](https://network.pivotal.io/products/pivotal-container-service/#/releases/386533/file_groups/1830).
 - [ ] uaac - this is generally installed as a Ruby gem 
> `gem install cf-uaac`
 - [ ] Curl
 - [ ]  jq
 - [ ] om - The required binary can be deployed [here](https://github.com/pivotal-cf/om/releases)


 - PKS plans with atleast one plan that has the ability to deploy K8s clusters with `Allow privilaged` setting enabled.
 - To deploy a K8s cluster use the following steps - 
 
```bash
$ UAA_PWD=`om -t [opsman_fqdn] -u [opsman_userid] -p ca271a1q1qapx1wc -k credentials --product-name pivotal-container-service --credential-reference .properties.pks_uaa_management_admin_client -t json|jq -r .secret`

$ echo $UAA_PWD
MgHXgeuYBaTddbAuY1bklOmf3PV-nCZ-

$ uaac token client get admin -s $UAA_PWD
Unknown key: Max-Age = 86400

Successfully fetched token via client credentials grant.
Target: https://api.pks.caracas.cf-app.com:8443
Context: admin, from client admin

$ uaac user add nverma --emails nverma@pivotal.io -p Passw0rd
user account successfully added

$ uaac member add pks.clusters.admin nverma
success

$ pks login -a api.pks.caracas.cf-app.com -u nverma -k

Password: ********
API Endpoint: api.pks.caracas.cf-app.com
User: nverma

$ pks plans

Name    ID                                    Description
small   8A0E21A8-8072-4D80-B365-D1F502085560  Example: This plan will configure a lightweight kubernetes cluster. Not recommended for production workloads.
medium  58375a45-17f7-4291-acf1-455bfdc8e371  Example: This plan will configure a medium sized kubernetes cluster, suitable for more pods.
large   241118e5-69b2-4ef9-b47f-4d2ab071aff5  Example: This plan will configure a large kubernetes cluster for resource heavy workloads, or a high number of workloads.
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc2NjYxNzk5NCwtOTAwMzIyNjQ1LC0yMD
c3NTczMzg0LC0xOTAyMTQ0MDkxLC03ODgwNjc2MjIsMjIwNTUz
NjIzXX0=
-->
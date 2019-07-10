
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


 - PKS plans with atleast one plan that has the ability to deploy K8s clusters with `Allow privilaged` setting enabled.
 - 


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM0NzA3MTE5NCwtNzg4MDY3NjIyLDIyMD
U1MzYyM119
-->
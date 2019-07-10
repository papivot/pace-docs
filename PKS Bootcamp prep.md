
The following artifacts needs to be available before the start of the bootcamp -

 - Ops Manager with PKS installed and configured. The recommended option is GCP as it is easy to deploy, manage and the lease expensive of the public clouds. Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Toolsmith has a help video on how to request a PKS environment in GCP. 
 - BOSH Director should have the `Enable VM Resurrector Plugin` already enabled.
 - Jumpbox/bastion host with access to the PKS environment (preferably Ubuntu Linux) with the following binaries installed -
		- Kubectl - Latest binaries can be downloaded from [here](https://storage.googleapis.com/kubernetes-release/release/$%28curl%20-s%20https://storage.googleapis.com/kubernetes-release/release/stable.txt%29/bin/linux/amd64/kubectl)
		- pks
		- jq
		- uaac
		- curl
 - PKS plans with atleast one plan that has the ability to deploy K8s clusters with `Allow privilaged` setting enabled.
 - 


<!--stackedit_data:
eyJoaXN0b3J5IjpbMjIwNTUzNjIzXX0=
-->
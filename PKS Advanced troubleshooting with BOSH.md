
# Advanced troubleshooting with BOSH

Besides the normal troubleshooting that can be performed using the Opsman UI and the kubectl commands, there are numerous troubleshooting processes that can be performed thru the BOSH CLI. 

### Login to BOSH Director

The best and the easiest way to login to the BOSH director is to login via the Ops Manager.  To complete the process -

 -  Navigate to the Pivotal Ops Manager FQDN in your browser.
 - Login.
 - Click on the Pivotal Director Tile (vSphere). 
 - Click the tab labeled  `status`. Here is the list of VMs deployed by the platform and the current status.
 - Note the IP of the  `Ops Manager Director`  job down, this is the Director. The director has the knowledge of all kubernetes clusters deployed.
 - Click the tab labeled  `credentials`. Here is the Pivotal Director credentials that the platform auto generates when deploying tiles.
 - Click  `Link to Credential`  under Director Credentials.
 - Note the  `identity`, and  `password`  down. This is the username and password we will use to connect to the Director.
 - Using SSH and the password set when deploying Pivotal Ops Manager, SSH to the Ops Manager VM. You can SSH into the Ops Manager machine using your favorite IaaS supported method. 
 
 > `ssh ubuntu@<PIVOTAL-OPS-MANAGER-FQDN`

In this example, you can use the `gcloud` command and authentication to SSH to your Ops Manager VM.

> `gcloud compute --project "[projectname]" ssh --zone "us-east1-c" "[opsman_vm_instance_name]"`

- Use the `bosh` cli to connect to the Director using the IP found earlier.
> `bosh alias-env pcf -e <DIRECTOR-IP> --ca-cert /var/tempest/workspaces/default/root_ca_certificate`

- Login to the `bosh` cli using the username, and password found earlier.
>`bosh -e pcf login`

- An output similar to this indicates a successful login 
```shell
Using environment '10.0.0.5'
 
Email (): director
Password ():

Successfully authenticated with UAA

Succeeded
```

### Viewing BOSH deployments 

BOSH deploys our PKS clusters. Let us list those deployments.

>`bosh -e pcf deployments`

This should reu
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NjgxNzczNzUsLTE1ODE5ODc5NDQsLT
E4NDM0NTIwMjddfQ==
-->
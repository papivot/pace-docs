
# Advanced troubleshooting with BOSH

Besides the normal troubleshooting that can be performed using the Opsman UI and the kubectl commands, there are numerous troubleshooting processes that can be performed thru the BOSH CLI. 

### Login to BOSH Director

The best and the easiest way to login to the BOSH director is to login via the Opsman. You can SSH into the Opsman machine using your favorite IaaS supported method. In this example, you can use the `gcloud` command and authentication to SSH to your Opsman VM.

1.  Navigate to the Pivotal Ops Manager FQDN.
2.  Login.
    
3.  Click on the Pivotal Director Tile (vSphere).
    
4.  Click the tab labeled  `status`. Here is the list of VMs deployed by the platform and the current status.
    
5.  Note the IP of the  `Ops Manager Director`  job down, this is the Director. The director has the knowledge of all kubernetes clusters deployed.
    
6.  Click the tab labeled  `credentials`. Here is the Pivotal Director credentials that the platform auto generates when deploying tiles.
    
7.  Click  `Link to Credential`  under Director Credentials.
    
8.  Note the  `identity`, and  `password`  down. This is the username and password we will use to connect to the Director.

> `gcloud compute --project "[projectname]" ssh --zone "us-east1-c" "[opsman_vm_instance_name]"`

Once logged in, execute the following commands to login to the BOSH director -

>

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc1MzY0MzI2Nl19
-->
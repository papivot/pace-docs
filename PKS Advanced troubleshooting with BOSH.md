
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

This should return something similar to this - 
```shell
Using environment '10.0.0.5' as user 'director'

Name                                                   Release(s)                            Stemcell(s)                                    Team(s)
pivotal-container-service-6a9d18c28f11275cff0b         backup-and-restore-sdk/1.8.0          bosh-google-kvm-ubuntu-xenial-go_agent/250.56  -
                                                       bosh-dns/1.10.0
                                                       bpm/1.0.4
                                                       cf-mysql/36.14.0.1
                                                       cfcr-etcd/1.10.0
                                                       docker/35.1.0
                                                       kubo/0.31.0
                                                       kubo-service-adapter/1.4.0-build.194
                                                       ...
                                                       
service-instance_47267983-9989-41ce-bc33-905e249b1fbc  bosh-dns/1.10.0                       bosh-google-kvm-ubuntu-xenial-go_agent/250.56  pivotal-container-service-6a9d18c28f11275cff0b
                                                       bpm/1.0.4
                                                       cfcr-etcd/1.10.0
                                                       docker/35.1.0
                                                       kubo/0.31.0
                                                       nsx-cf-cni/2.4.0.12511604
                                                       ...

2 deployments

Succeeded
```

-   Notice the  `Pivotal-container-service`  deployment which was deployed from the PKS tile. Also notice the  `service-instance_*`  which is our deployed PKS cluster, note it down.

Most of the commands, while troubleshooting, would need to reference the `service-instance-*` deployment. 

### SSH into K8s node VMs

Using the service-instance value in the previous command, you can execute the following command to list all the VMs associated with that particular deployment. 
>`bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc vms`

This should return an output similar to this - 

```shell
sing environment '10.0.0.5' as user 'director'

Task 333. Done

Deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc'

Instance                                     Process State  AZ          IPs        VM CID                                   VM Type      Active
master/38ed427f-2998-4de0-8906-50e0accb8ae1  running        us-east1-b  10.0.11.5  vm-f07c89ad-57ed-494b-6267-8ecab37f56ec  medium.disk  true
worker/29e4a450-fbfc-404e-aa92-772f678e1693  running        us-east1-b  10.0.11.6  vm-4376c6a8-fcfb-41a2-6ee9-4093de299e97  medium.disk  true
worker/c6ac73c5-7911-4eb9-bfe0-0b66a148f643  running        us-east1-d  10.0.11.8  vm-0d33fa8f-9acc-444c-50b8-afec139f933f  medium.disk  true
worker/d2ff055d-803d-44bc-af54-66b84dc1ebf5  running        us-east1-c  10.0.11.7  vm-ff5ccd32-6fd8-43f3-6849-9d56ba679784  medium.disk  true

4 vms

Succeeded
```

Note the Instance ID and the VM CID. Use the Instance ID to ssh into the required VM. 

>```shell
>bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc vms ssh master/38ed427f-2998-4de0-8906-50e0accb8ae1
>```


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk3ODQ0NzcxNCwtMTU4MTk4Nzk0NCwtMT
g0MzQ1MjAyN119
-->
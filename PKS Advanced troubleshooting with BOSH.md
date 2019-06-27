
# PKS Advanced troubleshooting with BOSH

### Requirements 
- Access to Pivotal Ops Manager UI
- SSH Access to Pivotal Ops Manager

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
>bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc ssh master/38ed427f-2998-4de0-8906-50e0accb8ae1
>```

This should return something to this - 

```shell
sing environment '10.0.0.5' as user 'director'

Using deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc'

Task 334. Done
Unauthorized use is strictly prohibited. All access and activity
is subject to logging and monitoring.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-50-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Fri Jun 21 14:59:13 2019 from 10.0.0.4
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

master/38ed427f-2998-4de0-8906-50e0accb8ae1:~$
```

Congratulations !!! You have logged into a BOSH deployed K8s node.

### SCP files into K8s VMs

At times you may need to transfer some files to the K8s nodes that are managed by BOSH. This can be achieved similar to the BOSH CLI ssh command. Execute the following command to copy a file [.bash_history] to the /tmp folder in one of the master nodes. 

> `bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc scp .bash_history master/38ed427f-2998-4de0-8906-50e0accb8ae1:/tmp`

```shell
Using environment '10.0.0.5' as user 'director'

Using deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc'

Task 338. Done
master/38ed427f-2998-4de0-8906-50e0accb8ae1: stderr | Unauthorized use is strictly prohibited. All access and activity
master/38ed427f-2998-4de0-8906-50e0accb8ae1: stderr | is subject to logging and monitoring.

Succeeded
```
Once successfully completed, login and validate that the .bash_history file exist in /tmp of the master VM. 

### View tasks, events and logs

#### Tasks

 - To view the last 10 (example) tasks that were executed for the deployment, execute the following - 

>`bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc tasks --recent=10`

This should return something similar to this - 

```shell
332  done   Fri Jun 21 07:00:01 UTC 2019  Fri Jun 21 07:00:01 UTC 2019  scheduler                                       service-instance_47267983-9989-41ce-bc33-905e249b1fbc  snapshot deployment                                                                                      snapshots of deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc' created
328  done   Thu Jun 20 07:00:02 UTC 2019  Thu Jun 20 07:00:02 UTC 2019  scheduler                                       service-instance_47267983-9989-41ce-bc33-905e249b1fbc  snapshot deployment                                                                                      snapshots of deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc' created
322  done   Wed Jun 19 07:00:01 UTC 2019  Wed Jun 19 07:00:01 UTC 2019  scheduler                                       service-instance_47267983-9989-41ce-bc33-905e249b1fbc  snapshot deployment                                                                                      snapshots of deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc' created
316  done   Tue Jun 18 22:00:31 UTC 2019  Tue Jun 18 22:03:03 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand telemetry-agent from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc         1 succeeded, 0 errored, 0 canceled
314  done   Tue Jun 18 21:59:31 UTC 2019  Tue Jun 18 21:59:36 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand vrops-errand from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc            3 succeeded, 0 errored, 0 canceled
313  done   Tue Jun 18 21:55:30 UTC 2019  Tue Jun 18 21:58:41 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand wavefront-proxy-errand from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc  1 succeeded, 0 errored, 0 canceled
312  done   Tue Jun 18 21:52:30 UTC 2019  Tue Jun 18 21:55:20 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand apply-addons from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc            1 succeeded, 0 errored, 0 canceled
311  done   Tue Jun 18 21:50:29 UTC 2019  Tue Jun 18 21:52:25 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  create deployment                                                                                        /deployments/service-instance_47267983-9989-41ce-bc33-905e249b1fbc
287  done   Tue Jun 18 19:41:10 UTC 2019  Tue Jun 18 19:44:15 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand telemetry-agent from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc         1 succeeded, 0 errored, 0 canceled
286  done   Tue Jun 18 19:40:10 UTC 2019  Tue Jun 18 19:40:14 UTC 2019  pivotal-container-service-6a9d18c28f11275cff0b  service-instance_47267983-9989-41ce-bc33-905e249b1fbc  run errand vrops-errand from deployment service-instance_47267983-9989-41ce-bc33-905e249b1fbc            3 succeeded, 0 errored, 0 canceled

10 tasks

Succeeded
```

The output also lists successes, errors and cancellations. This is very helpful for troubleshooting. 

 - To view what actions were preformed for a specific task, execute the following (in this example 311 is the task_ID that we need more information on)

>`bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc task -a 311`

This should return details on that task id - 
```shell
Using environment '10.0.0.5' as user 'director'

Task 311

Task 311 | 21:50:29 | Preparing deployment: Preparing deployment
Task 311 | 21:50:31 | Warning: DNS address not available for the link provider instance: pivotal-container-service/495a5f24-03fc-4f4b-b816-aab0ad40b5d3
Task 311 | 21:50:31 | Warning: DNS address not available for the link provider instance: pivotal-container-service/495a5f24-03fc-4f4b-b816-aab0ad40b5d3
Task 311 | 21:50:31 | Warning: DNS address not available for the link provider instance: pivotal-container-service/495a5f24-03fc-4f4b-b816-aab0ad40b5d3
...
Task 311 Started  Tue Jun 18 21:50:29 UTC 2019
Task 311 Finished Tue Jun 18 21:52:25 UTC 2019
Task 311 Duration 00:01:56
Task 311 done

Succeeded
```
#### Events

- To fetch all events that were generated for the deployment, execute the following

> `bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc events`

This should return something similar to this - 
```shell
Using environment '10.0.0.5' as user 'director'

ID            Time                          User                                            Action       Object Type  Object Name                                                                                     Task ID  Deployment                                             Instance                                               Context                                                                                               Error
4180          Fri Jun 21 17:14:50 UTC 2019  director                                        cleanup ssh  instance     master/38ed427f-2998-4de0-8906-50e0accb8ae1                                                     341      service-instance_47267983-9989-41ce-bc33-905e249b1fbc  master/38ed427f-2998-4de0-8906-50e0accb8ae1            user: ^bosh_196c48197974477                                                                           -
4179          Fri Jun 21 17:11:39 UTC 2019  director                                        setup ssh    instance     master/38ed427f-2998-4de0-8906-50e0accb8ae1                                                     340      service-instance_47267983-9989-41ce-bc33-905e249b1fbc  master/38ed427f-2998-4de0-8906-50e0accb8ae1            user: bosh_196c48197974477                                                                            -
4178          Fri Jun 21 15:52:01 UTC 2019  director                                        cleanup ssh  instance     master/38ed427f-2998-4de0-8906-50e0accb8ae1                                                     339      service-instance_47267983-9989-41ce-bc33-905e249b1fbc  master/38ed427f-2998-4de0-8906-50e0accb8ae1            user: ^bosh_851662ac898b4b0                                                                           -
4177          Fri Jun 21 15:51:59 UTC 2019  director                                        setup ssh    instance     master/38ed427f-2998-4de0-8906-50e0accb8ae1                                                     338      service-instance_47267983-9989-41ce-bc33-905e249b1fbc  master/38ed427f-2998-4de0-8906-50e0accb8ae1            user: bosh_851662ac898b4b0                                                                            -
...
```

To view details of an event, execute the event command with the event_id as a parameter. 

> `bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc event 4180`

should retune something similar to this - 

```shell
Using environment '10.0.0.5' as user 'director'

ID           4180
Time         Fri Jun 21 17:14:50 UTC 2019
User         director
Action       cleanup ssh
Object Type  instance
Object Name  master/38ed427f-2998-4de0-8906-50e0accb8ae1
Task ID      341
Deployment   service-instance_47267983-9989-41ce-bc33-905e249b1fbc
Instance     master/38ed427f-2998-4de0-8906-50e0accb8ae1
Context      user: ^bosh_196c48197974477

Succeeded
```
#### Logs

- To fetch logs from **all** the VMs associated with the deployment, execute the following - 
> `bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc logs`

This should return something similar to this output  - 
```shell
Using environment '10.0.0.5' as user 'director'

Using deployment 'service-instance_47267983-9989-41ce-bc33-905e249b1fbc'

Task 336

Task 336 | 15:16:32 | Fetching logs for worker/c6ac73c5-7911-4eb9-bfe0-0b66a148f643 (2): Finding and packing log files
Task 336 | 15:16:32 | Fetching logs for worker/d2ff055d-803d-44bc-af54-66b84dc1ebf5 (1): Finding and packing log files
Task 336 | 15:16:32 | Fetching logs for master/38ed427f-2998-4de0-8906-50e0accb8ae1 (0): Finding and packing log files
Task 336 | 15:16:32 | Fetching logs for worker/29e4a450-fbfc-404e-aa92-772f678e1693 (0): Finding and packing log files (00:00:01)
Task 336 | 15:16:33 | Fetching logs for worker/c6ac73c5-7911-4eb9-bfe0-0b66a148f643 (2): Finding and packing log files (00:00:01)
Task 336 | 15:16:33 | Fetching logs for worker/d2ff055d-803d-44bc-af54-66b84dc1ebf5 (1): Finding and packing log files (00:00:01)
Task 336 | 15:16:43 | Fetching logs for master/38ed427f-2998-4de0-8906-50e0accb8ae1 (0): Finding and packing log files (00:00:11)
Task 336 | 15:16:44 | Fetching group of logs: Packing log files together

...
Downloading resource '9af4ae05-c056-44c5-b582-2d5613fb6e54' to '/home/nverma/service-instance_47267983-9989-41ce-bc33-905e249b1fbc-20190621-151646-585885543.tgz'...

################################                              52.53% 81.50 MiB/s
Succeeded
```

Note that a tgz file gets downloaded to the user home directory in Ops Manager VM. This extracted files can be very valuable for further troubleshooting and analysis. This can be also used by Pivotal Support.

### Deleting a cluster deployment 

While all cluster deployments have to be created and managed using the PKS CLI, it may happen at times that there is a failure during the creation of K8S clusters using the PKS CLI. This can happen for multiple reasons (outside the scope of this excecise) but may leave the deployment in a a funky state. PKS CLI may not be able to delete the cluster using `pks delete-cluster [cluster_name]` command. 

To delete such a deployment, you can leverage the BOSH CLI to clean up, similar to this example -

> `bosh -e pcf -d service-instance_47267983-9989-41ce-bc33-905e249b1fbc delete-deployment`

Once done, clean up the reference in PKS database by execute 

> `pks delete-cluster [cluster_name]` 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg3MzE1MzcxMiw3Mzc1ODA4NDksNTczNj
Y0NTM3LDIwOTY0MjQxNDYsMTI4NjgxMDkwMCwtMTE5MTk0OTE5
LDU4Nzk2Mjc4NCwtMjE0OTExMDksLTcwMzIxMzIyNCwtNjc2NT
A4NTAyLDEwNTYyNTk2MDUsMTczMDgxMDEsLTE1ODE5ODc5NDQs
LTE4NDM0NTIwMjddfQ==
-->
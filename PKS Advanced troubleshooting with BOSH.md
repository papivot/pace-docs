
# Advanced troubleshooting with BOSH

Besides the normal troubleshooting that can be performed using the Opsman UI and the kubectl commands, there are numerous troubleshooting processes that can be performed thru the BOSH CLI. 

### Login to BOSH Director

The best and the easiest way to login to the BOSH director is to login via the Opsman. You can SSH into the Opsman machine using your favorite IaaS supported method. In this example, you can use the `gcloud` command to SSH to your Opsman VM.

> `gcloud compute --project "pa-nverma" ssh --zone "us-east1-c" "gcpcloud-ops-manager"
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMwMTE2Mzc3OF19
-->
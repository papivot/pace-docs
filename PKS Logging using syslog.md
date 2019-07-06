
# PKS Logging using syslog

PKS logging can be enabled and configured at 3 different levels - 

1. Individual K8s namespaces within a cluster
2. Individual K8s clusters
3. PKS Control plane
4. BOSH Director and its components

For this lab, we will use Papertrail. Users can bring their own logging systems that follow the syslog protocol. 

The following information will be required - 

* Syslog endpoint - `logs3.papertrailapp.com` for this lab
* Syslog port - `39458` for this lab
* TLS enabled/disabled - `Enabled` for this lab. 
* `Enable Log Sink Resource` should be enabled on the PKS tile.

## PKS Control plane

To enable logging of the PKS control plane, within Opsman UI, navigate to the PKS tile -> `Settings` -> `Logging`.

The following information needs to be input - 

`Enable Syslog for PKS?` - `Yes`
`Address` -`logs3.papertrailapp.com` 
`Port` - `39458`
`Transport Protocol` - `TCP`
`Enable TLS` - Checked if the logging app allows TLS.

`Save` the changes. 

Navigate to the top level. `Review Pending Changes` and then `Apply Changes`. 

Look for relevant logs in your Syslog dashboard

## BOSH Director and its components

To enable logging of the BOSH director and its components, within Opsman UI, navigate to the Bosh Director tile -> `Settings` -> `Syslog`.

The following information needs to be input - 

`Do you want to configure Syslog for Bosh Director?` - `Yes`
`Address` -`logs3.papertrailapp.com` 
`Port` - `39458`
`Transport Protocol` - `TCP`
`Enable TLS` - Checked if the logging app allows TLS.

`Save Syslog Settings` the changes. 

Navigate to the top level. `Review Pending Changes` and then `Apply Changes`. 

Look for relevant logs in your Syslog dashboard
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQzODA0MDEwMywtMjEzMTc0ODMwOSwyMD
A5NDM5NTU1LDc4NTY5NjA1NywyMTA2OTE0ODQ3LDEyMjY5ODIz
ODldfQ==
-->
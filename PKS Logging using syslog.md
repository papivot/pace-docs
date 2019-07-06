
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

To enable logging of the PKS control plane, within Opsman UI, navigate to the PKS tile -> Settings -> Logging.

The following information needs to be input - 

`Enable Syslog for PKS?` - `Yes`
`Address` -`logs3.papertrailapp.com` 
`Port` - `39458`
`Transport Protocol` - `TCP`

Save the changes. 

Navigate to the top level. Review Pending changes and then Apply Chnages. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzg1Njk2MDU3LDIxMDY5MTQ4NDcsMTIyNj
k4MjM4OV19
-->
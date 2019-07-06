
# PKS Logging using syslog

PKS logging can be enabled and configured at 3 different levels - 

1. Individual K8s namespaces within a cluster
2. Individual K8s clusters
3. PKS Control plane
4. BOSH Director and its components

For this lab, we will use [Papertrail](https://papertrailapp.com) syslog application . Users can bring their own logging systems that follow the syslog protocol. 

The following information will be required - 

* Syslog endpoint - `logs3.papertrailapp.com` for this lab
* Syslog port - `39458` for this lab
* TLS enabled/disabled - `Enabled` for this lab. 
* `Enable Log Sink Resource` should be enabled on the PKS tile.

## Individual K8s namespaces within a cluster

Create a yaml file `sink.yaml` with the following contents -
```yaml
apiVersion: apps.pivotal.io/v1beta1
kind: Sink
metadata:
  name: papertrail-sink
  namespace: kube-system
spec:
  type: syslog
  host: logs3.papertrailapp.com
  port: 39458
  enable_tls: true
```
where the `name`  is the name  of your Sink resource, `host` is the syslog host, `port` is the syslog port and `enable_tls` enables TLS communication.

> `kubectl apply -f sink.yaml`

This should create the required ClusterSink resource. 

## Individual K8S clusters

Create a yaml file `clustersink.yaml` with the following contents -

```yaml
apiVersion: apps.pivotal.io/v1beta1
kind: ClusterSink
metadata:
   name: papertrail-clustersink
spec:
   type: syslog
   host: logs3.papertrailapp.com
   port: 39458
   enable_tls: true
```

where the `name`  is the name  of your ClusterSink, `host` is the syslog host, `port` is the syslog port and `enable_tls` enables TLS communication. 

> `kubectl apply -f clustersink.yaml`

This should create the required ClusterSink resource. 

Navigate to your Syslog dashboard and look for relevant logs in your Syslog dashboard

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

Navigate to your Syslog dashboard and look for relevant logs in your Syslog dashboard.

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

Navigate to your Syslog dashboard and look for relevant logs in your Syslog dashboard.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk0MDMzNDA0MywtMTQwMjk2MjU1OSwtMj
EzMTc0ODMwOSwyMDA5NDM5NTU1LDc4NTY5NjA1NywyMTA2OTE0
ODQ3LDEyMjY5ODIzODldfQ==
-->
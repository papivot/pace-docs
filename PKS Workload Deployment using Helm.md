
# PKS Workload Deployment using Helm

In this lab/demo, we will show you how to deploy a set of applications using Helm on a K8s cluster. This exercise serves multiple purposes - 

1. How to interact with Helm to deploy applications
2. Since the choice of application is Prometheus and Grafana, it will also show how to start with monitoring and reporting on a K8s cluster
3. Show how to interact with an application running on K8s cluster. 

### Requirements 
- A working K8s cluster with admin access. (*Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Upon a PKS environment has been created, you can login using the pks cli and create a cluster using an active plan.)*

## Helm Deployment

To deploy the CLI, use brew on Mac-

> `brew install kubernetes-helm`

Helm has a client server model where the server component - **Tiller** -  runs on the Kubernetes cluster where helm targets the deployment. Helm uses the kubeconfig file to connect to the server component running on the K8s cluster. 

> `helm version`

Without a valid kubeconfig, you get this - 

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Error: Get http://localhost:8080/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller: dial tcp [::1]:8080: connect: connection refused
```
With a valid kubeconfig, where helm has not been initialized, you will get this -

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Error: could not find a ready tiller pod
```

Before installing the Tiller server component, we need to create a service account and give it some elevated privileges. Use the following yaml - 

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

> `kubectl apply -f rbac-config.yaml`

where rbac-config.yaml is the name of the file with the above contents. Once completed, perform helm init to deploy the Tiller server component. 

> `helm init --service-account tiller`

This will install the server component. `kubectl get pods -n kube-system` should give something similar to this - 

```shell
NAME                                    READY   STATUS    RESTARTS   AGE
...
tiller-deploy-6d65d78679-p5cm9          1/1     Running   0          128m
...
```

> `helm version`

With a valid kubeconfig to the cluster and tiller installed, you get something similar to this -

```shell
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
```

You are now ready to use helm to deploy applications in the cluster. 

## Prometheus Deployment

Before we start with the deployment, we will create a namespace called **monitoring** and limit all work to the monitoring namespace.

> `kubectl create namespace monitoring`

To search for the Prometheus charts, use the command - 

> `helm search prometheus`

which should return something similar to this - 

```shell
NAME                                 	CHART VERSION	APP VERSION	DESCRIPTION
stable/prometheus                    	8.14.0       	2.11.1     	Prometheus is a monitoring system and time series database.
stable/prometheus-adapter            	1.2.0        	v0.5.0     	A Helm chart for k8s prometheus adapter
stable/prometheus-blackbox-exporter  	0.4.0        	0.14.0     	Prometheus Blackbox Exporter
...
stable/prometheus-nats-exporter      	2.1.0        	0.4.0      	A Helm chart for prometheus-nats-exporter
stable/prometheus-node-exporter      	1.5.1        	0.18.0     	A Helm chart for prometheus node-exporter
...
```

We will use the `stable/prometheus` chart for this session. 

Use the inspect command to get details on the chart - 

> `helm inspect stable/prometheus`

This should list a detailed output.

```shell
...
## TL;DR;

```console
$ helm install stable/prometheus

## Introduction

This chart bootstraps a [Prometheus](https://prometheus.io/) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.3+ with Beta APIs enabled

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm install --name my-release stable/prometheus
...
```

Use the following command to install prometheus and its required components - 

> `helm install stable/prometheus --namespace monitoring --name prometheus`

This should create the necessary artifacts and start the pods. The output will detail what was created as part of this install. 

```shell
NAME:   prometheus
LAST DEPLOYED: Fri Jun 28 15:01:40 2019
NAMESPACE: monitoring
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
prometheus-alertmanager  1     1s
prometheus-server        3     1s

==> v1/PersistentVolumeClaim
NAME                     STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
prometheus-alertmanager  Pending  1s
prometheus-server        Pending  1s

==> v1/Pod(related)
NAME                                            READY  STATUS             RESTARTS  AGE
prometheus-alertmanager-668785dc56-d7bbm        0/2    Pending            0         1s
prometheus-kube-state-metrics-7bcf787fd4-lt2rf  0/1    ContainerCreating  0         1s
prometheus-node-exporter-9gfwp                  0/1    ContainerCreating  0         1s
prometheus-node-exporter-svxng                  0/1    ContainerCreating  0         1s
prometheus-node-exporter-sw744                  0/1    ContainerCreating  0         1s
prometheus-pushgateway-75dc77db76-4pjzj         0/1    ContainerCreating  0         1s
prometheus-server-5d5f6db7cc-mkpwg              0/2    Pending            0         1s

==> v1/Service
NAME                           TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
prometheus-alertmanager        ClusterIP  10.100.200.147  <none>       80/TCP    1s
prometheus-kube-state-metrics  ClusterIP  None            <none>       80/TCP    1s
prometheus-node-exporter       ClusterIP  None            <none>       9100/TCP  1s
prometheus-pushgateway         ClusterIP  10.100.200.107  <none>       9091/TCP  1s
prometheus-server              ClusterIP  10.100.200.171  <none>       80/TCP    1s

==> v1/ServiceAccount
NAME                           SECRETS  AGE
prometheus-alertmanager        1        1s
prometheus-kube-state-metrics  1        1s
prometheus-node-exporter       1        1s
prometheus-pushgateway         1        1s
prometheus-server              1        1s

==> v1beta1/ClusterRole
NAME                           AGE
prometheus-kube-state-metrics  1s
prometheus-server              1s

==> v1beta1/ClusterRoleBinding
NAME                           AGE
prometheus-kube-state-metrics  1s
prometheus-server              1s

==> v1beta1/DaemonSet
NAME                      DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
prometheus-node-exporter  3        3        0      3           0          <none>         1s

==> v1beta1/Deployment
NAME                           READY  UP-TO-DATE  AVAILABLE  AGE
prometheus-alertmanager        0/1    1           0          1s
prometheus-kube-state-metrics  0/1    1           0          1s
prometheus-pushgateway         0/1    1           0          1s
prometheus-server              0/1    1           0          1s
...
```

Run the following to get the pod status - 

> `kubectl get pods -n monitoring`

which should display something like this - 

```shell
NAME                                             READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-668785dc56-d7bbm         0/2     Pending   0          2m36s
prometheus-kube-state-metrics-7bcf787fd4-lt2rf   1/1     Running   0          2m36s
prometheus-node-exporter-9gfwp                   1/1     Running   0          2m36s
prometheus-node-exporter-svxng                   1/1     Running   0          2m36s
prometheus-node-exporter-sw744                   1/1     Running   0          2m36s
prometheus-pushgateway-75dc77db76-4pjzj          1/1     Running   0          2m36s
prometheus-server-5d5f6db7cc-mkpwg               0/2     Pending   0          2m36s
```

Notice that it may happen that some of the pods are in a pending state. To troubleshoot, run the following - 

> `kubectl describe pod prometheus-server-5d5f6db7cc-mkpwg -n monitoring` 

where `prometheus-server-5d5f6db7cc-mkpwg` is the name of the pod from the previous output.  This should provide a status and a possible reason for failure - 

```shell
...
Events:
  Type     Reason            Age                   From               Message
  ----     ------            ----                  ----               -------
  Warning  FailedScheduling  24s (x10 over 4m37s)  default-scheduler  pod has unbound immediate PersistentVolumeClaims (repeated 3 times)
```

The pod is looking for some storage in the form of PersistantVolumeClaim and is unable to get it. Similar issues were discovered for alertmanager. We will have to provide it in the next few steps. 

Delete the prometheus release -

> `helm ls`

> `helm delete prometheus --purge`

#### Creating a storage class

If not already available, we will need to create a storage class. This is specific to the infrastructure. Some of the supported storage classes are provided [here](https://kubernetes.io/docs/concepts/storage/storage-classes/)

Here is a simple sample yaml that we will create for AWS to use the EBS volume - 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: awsgp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
```

A similar one for GCP could be - 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gcepd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
```

For vSphere, it could be - 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: vmdisk
provisioner: kubernetes.io/vsphere-volume
parameters:
    datastore: [DATASTORENAME]
    diskformat: thin
    fstype: ext3
```

Use kubectl to apply the yaml to create the StorageClass - 

> `kubectl apply -f storageclass.yaml`

#### Start prometheus with the correct values

Inspect the chart to view the parameters using `helm inspect ...`. Both alertmanager and prometheus have PersistantVolume enabled where the storageClass is set to null. 

```shell
`alertmanager.persistentVolume.enabled` | If true, alertmanager will create a Persistent Volume Claim | `true`
...
`alertmanager.persistentVolume.storageClass` | alertmanager data Persistent Volume Storage Class | `unset`
...
`server.persistentVolume.enabled` | If true, Prometheus server will create a Persistent Volume Claim | `true`
...
`server.persistentVolume.storageClass` | Prometheus server data Persistent Volume Storage Class |  `unset`
```

Deploy the chart with the correct values in the CLI - 

> `helm install stable/prometheus --namespace monitoring --name prometheus --set alertmanager.persistentVolume.storageClass=awsgp2,server.persistentVolume.storageClass=awsgp2`

This should do the trick. Within a few minutes, validate all pods are up and running. 

> `kubectl get pods -n monitoring`

should return something similar to - 

```bash
NAME                                             READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-668785dc56-f896b         1/2     Running   0          41s
prometheus-kube-state-metrics-7bcf787fd4-nbk7p   1/1     Running   0          41s
prometheus-node-exporter-l9shp                   1/1     Running   0          41s
prometheus-node-exporter-n8fzf                   1/1     Running   0          41s
prometheus-node-exporter-s46b2                   1/1     Running   0          41s
prometheus-pushgateway-75dc77db76-w4g69          1/1     Running   0          41s
prometheus-server-5d5f6db7cc-mq2gp               1/2     Running   0          41s
```

All the required components of Prometheus have now started and collecting and storing data.

## Grafana Deployment

Before we deploy the pod, we will create a configmap that contains the definition from where Grafana needs to use the datasource - 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-grafana-datasource
  namespace: monitoring
  labels:
    grafana_datasource: '1'
data:
  datasource.yaml: |-
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      access: proxy
      orgId: 1
      url: http://prometheus-server.monitoring.svc.cluster.local
```

> `kubectl apply -f grafanaconfigmap.yaml`

where `grafanaconfigmap.yaml` contains the above yaml data. 

Use `helm search` and `helm inspect` to get details on the Grafana charts. Use the following to deploy the chart - 

> `helm install stable/grafana --namespace monitoring --name grafana --set sidecar.datasources.enabled=true`

This should create the required artifacts and start the pods. 

>`kubectl get pods -n monitoring`

```bash
grafana-666649ff75-zbmcf                         1/1     Running   0          44s
prometheus-alertmanager-668785dc56-f896b         2/2     Running   0          12m
prometheus-kube-state-metrics-7bcf787fd4-nbk7p   1/1     Running   0          12m
prometheus-node-exporter-l9shp                   1/1     Running   0          12m
prometheus-node-exporter-n8fzf                   1/1     Running   0          12m
prometheus-node-exporter-s46b2                   1/1     Running   0          12m
prometheus-pushgateway-75dc77db76-w4g69          1/1     Running   0          12m
prometheus-server-5d5f6db7cc-mq2gp               2/2     Running   0          12m
```

Congratulations! You have completed the deployment of Prometheus and Grafana. 

## Accessing the dashboard (application)

Get the Grafana admin credentials. 

> `kubectl get secret grafana --namespace monitoring -o json|jq -r '.data."admin-password"'|base64 --decode; echo`

should display a password string. Capture it as you will need it to login to the UI interface. 

#### Using ClusterIP

Since the current Grafana deployment is running with just service of type Cluster IP, we need to port forward using the kubectl command to get access to the UI. 

Use the first command to get the exact name of the pod and then the second command to execute the port forwarding. 

> `kubectl get pods -n monitoring |grep grafana`

> `kubectl port-forward grafana-666649ff75-zbmcf 3000 -n monitoring`

Using the browser navigate to `http://http://localhost:3000/` Use admin and the password that you retrieved earlier to login to the Grafana console. 

Once logged in, import a prebuilt dashboard - 

On the left panal, click on `+` -> `Import`. In the `Grafana.com dashboard` enter `1860` and click `Load`.

In the next screen, select `Prometheus` as the datasource and click `Import`  

You will now be able to view that metrics that the Node Exporter is collecting from each of the worker nodes and sending it up to Prometheus and then Grafana is visualizing the data!!!

#### Using LoadBalancer

Now, we will use the Service type LoadBalancer to access the Grafana UI. 

Lets destroy the Grafana deployment - 

> `helm delete grafana --purge`

> `helm inspect stable/grafana` 

shows there is a parameter called service.type that we will modify in this execution. 

> `helm install stable/grafana --namespace monitoring --name grafana --set sidecar.datasources.enabled=true,service.type=LoadBalancer`

Wait for a few minutes. A new LoadBalancer will be created in your IaaS, and you can access the Grafana UI using the load balancer IP/DNS. 

> `kubectl get service -n monitoring`

should return - 

```bash
NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
grafana                         LoadBalancer   10.100.200.98    a234d08d699e411e9867f024160e0f4d-298480210.us-east-2.elb.amazonaws.com   80:30759/TCP   7m55s
...
```

Browse to the external IP - `a234d08d699e411e9867f024160e0f4d-298480210.us-east-2.elb.amazonaws.com` in this example to access the UI

#### Using Ingress

Layer 7 (HTTP and HTTPS) ingress is provided by NSX-T PKS integration natively. For other IaaS and SDN, a separate Ingress controller needs to be installed on the cluster to enable Ingress routing. For the demo purpose, we will use **Traefik**, one of the leading Ingress controller providers. 

##### Install Traefik

We will use the following yaml to deploy the required artifacts to enable the Traefik ingress controller within our Kubernetes cluster. 

```yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
  labels:
    project.app: traefik-ingress
    project.name: traefik
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
  labels:
    project.app: traefik-ingress
    project.name: traefik
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
    namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    project.app: traefik-ingress
    project.name: traefik
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    project.app: traefik-ingress
    project.name: traefik
spec:
  selector:
    matchLabels:
      project.app: traefik-ingress
  template:
    metadata:
      labels:
        project.app: traefik-ingress
        project.name: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      dnsPolicy: ClusterFirstWithHostNet
      hostNetwork: true
      containers:
        - image: traefik
          name: traefik-ingress
          args:
            - --configfile=/config/traefik.toml
          resources:
            requests:
              cpu: 250m
              memory: 250Mi
            limits:
              cpu: 1
              memory: 2Gi
          readinessProbe:
            tcpSocket:
              port: 8080
            failureThreshold: 1
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            tcpSocket:
              port: 80
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 2
          securityContext:
            allowPrivilegeEscalation: false
          volumeMounts:
            - mountPath: /config
              name: config
              readOnly: true
          ports:
            - name: http
              containerPort: 80
              hostPort: 80
              protocol: TCP
            - name: httpn
              containerPort: 8880
              hostPort: 8880
              protocol: TCP
            - name: dash
              containerPort: 8080
              hostPort: 8080
              protocol: TCP
      volumes:
        - name: config
          configMap:
            name: traefik-toml
  updateStrategy:
    type: RollingUpdate
---
apiVersion: v1
data:
  traefik.toml: |
    # traefik.toml
    logLevel = "INFO"
    InsecureSkipVerify = true
    checkNewVersion = false
    defaultEntryPoints = ["http","https"]
    [entryPoints]
      [entryPoints.http]
        address = ":80"
        compress = true
      [entryPoints.httpn]
        address = ":8880"
        compress = true
      [entryPoints.traefik]
        address = ":8080"
    [ping]
      entryPoint = "http"
    [kubernetes]
      ingressClass = "traefik"
    [api]
      entryPoint = "traefik"
      dashboard = true
kind: ConfigMap
metadata:
  labels:
    project.app: traefik-ingress
    project.name: traefik
  name: traefik-toml
  namespace: kube-system
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-lb
  namespace: kube-system
  labels:
    project.app: traefik-ingress
    project.name: traefik
spec:
  selector:
    project.app: traefik-ingress
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      name: admin
    - protocol: TCP
      port: 80
      targetPort: 80
      name: web
    - protocol: TCP
      port: 443
      targetPort: 443
      name: ssl
  type: LoadBalancer
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: "PathPrefixStrip"
    ingress.kubernetes.io/custom-response-headers: "Strict-Transport-Security: max-age=31536000; includeSubDomains"
  labels:
    project.app: traefik-ingress
    project.name: traefik
spec:
  rules:
    - host: [[ cluster fqdn ]]
      http:
        paths:
          - path: /console
            backend:
              serviceName: traefik-ingress-lb
              servicePort: 8080
```

Copy and save the file as a yaml. Modify the `[[ cluster fqdn ]]` variable to meet your a cluster FQDN that you will use to access workloads - e.g. awscluster00.awscloud.navneetv.com. This is specific to your DNS environment. 

Apply the yaml file-

>`kubectl apply -f traefik.yml`

This should create all the required artifacts - 

```bash
clusterrole.rbac.authorization.k8s.io/traefik-ingress-controller created
clusterrolebinding.rbac.authorization.k8s.io/traefik-ingress-controller created
serviceaccount/traefik-ingress-controller created
daemonset.extensions/traefik-ingress-controller created
configmap/traefik-toml created
service/traefik-ingress-lb created
ingress.extensions/traefik-dashboard created
```

Once done, execute the following and wait for `EXTERNAL-IP` changes from `<pending>` to a final value - 

> `kubectl get svc traefik-ingress-lb -n kube-system -w`

Grab the `EXTERNAL-IP` and modify your DNS to reflect the `[[ cluster fqdn ]]`

Once done, navigate to http://`[[ cluster fqdn ]]`/console. The Traefik dashboard should open.

With the Traefik Ingress now controller working, it is now time to modify the Grafana access parameters. 

Apply this yaml file - grafana-ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: "PathPrefixStrip"
    ingress.kubernetes.io/custom-response-headers: "Strict-Transport-Security: max-age=31536000; includeSubDomains"
spec:
  rules:
    - host: grafana.[[ cluster fqdn ]]
      http:
        paths:
          - backend:
              serviceName: grafana
              servicePort: 80
```
Copy and save the file as a yaml. Modify the `[[ cluster fqdn ]]` variable to meet your a cluster FQDN that you will use to access workloads - eg awscluster00.awscloud.navneetv.com. This is specific to your DNS environment. 

>`kubectl apply -f grafana-ingress.yaml`

Check the ingress dashboard to verify the new config is in place. 

Edit the Grafana service and remove the existing load balancer configuration. >`kubectl edit service grafana -n monitoring`

Edit the `spec:` section to make it similar to this -

```yaml
spec:
  ports:
  - name: service
    port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana
    release: grafana
  sessionAffinity: None
  type: ClusterIP
```
 
Save and exit the vi editor (`:wq!`)

Browse to the external IP - `http://grafana.[[cluster fqdn]]/login`
to access your Grafana interface.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc1NzgzMTkxMywtMTE3MTE1OTYwMywxNz
AyNDE2NTk5LDY2ODUxNTMxOSwtMTQ5NTQ5ODUzLDIwOTAxODQ5
OTgsLTEyNzc0NDI1OTgsLTg3MTExNjA4NywtMTk0NzA0MDYzOC
wtMzc3MDg4NjUyLDE3MDc0MDg1NjEsMTYyMzU3NjU2MSwxNTM2
NjExMDA5LDE2OTAyNTkxMjMsLTExNTQ4NDY5NjQsLTEzMDI3Mz
g4OSw3NjExMzY0MzIsLTE4ODM4MTQ2ODMsOTMwODA2MDE1XX0=

-->
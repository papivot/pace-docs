
# PKS Workload Deployment using Helm

In this lab/demo, we will show you how to deploy a set of applications using yaml configuration files on a K8s cluster. This exercise serves multiple purposes - 

1. How to interact with yaml files to deploy applications
2. Show how to interact with an application running on K8s cluster. 

### Requirements 
- A working K8s cluster with admin access. (*Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Upon a PKS environment has been created, you can login using the pks cli and create a cluster using an active plan.)*

### Preparation

Login to the PKS API endpoint

>`pks login -k -a api.pks.domain.com -u username`                                                                                                 

```shell
Password: ********
API Endpoint: api.pks.domain.com
User: username
```

Get the list of clusters deployed - 

> `pks clusters`

should output the clusters deployed 

```shell
Name          Plan Name  UUID                                  Status     Action
gcpcluster00  small      69c4444c-8bf9-4907-956e-d94db7e145c2  succeeded  CREATE
```

>` pks get-credentials cluster_name`

will generate the kubeconfig file for the admin user `username` and allow kubectl commands to be executed against this cluster.

```shell
Fetching credentials for cluster gcpcluster00.
Context set for cluster gcpcluster00.

You can now switch between clusters by using:
$kubectl config use-context <cluster-name>
```

### Deploying the application

Copy the below yaml content and save it as a file call `k8sops-deployment.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kube-ops
  labels:
    project.name: k8s-operations
    project.app: k8s-operations
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: k8s-operations
  namespace: kube-ops
  labels:
    project.name: k8s-operations
    project.app: k8s-operations
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: k8s-operations
  labels:
    project.app: k8s-operations
    project.name: k8s-operations
rules:
  - apiGroups:
    - '*'
    resources:
    - '*'
    verbs: ["get", "list", "watch"]
  - nonResourceURLs:
    - '*'
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-operations
  labels:
    project.app: k8s-operations
    project.name: k8s-operations
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: k8s-operations
subjects:
- kind: ServiceAccount
  name: k8s-operations
  namespace: kube-ops
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  labels:
    project.app: k8s-operations
    project.name: k8s-operations
  name: k8s-operations
  namespace: kube-ops
spec:
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    project.app: k8s-operations
  type: LoadBalancer
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    project.name: k8s-operations
    project.app: k8s-operations
  name: k8s-operations
  namespace: kube-ops
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      project.app: k8s-operations
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        project.app: k8s-operations
    spec:
      containers:
      - env:
        - name: CLUSTER_NAME
          value: navneet.cfcr.demo
        image: whoami6443/k8soper:0.0.6
        imagePullPolicy: Always
        name: k8s-operations
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 64Mi
        securityContext:
          readOnlyRootFilesystem: true
          allowPrivilegeEscalation: false
          privileged: false
          runAsNonRoot: false
        volumeMounts:
        - mountPath: /user/k8soper
          name: cache-volume
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        stdin: true
        tty: true
      volumes:
      - name: cache-volume
        emptyDir: {}
      dnsPolicy: ClusterFirst
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: k8s-operations
      serviceAccountName: k8s-operations
      terminationGracePeriodSeconds: 30
```
Once saved, use kubectl to deploy the relevent K*

> `kubectl apply -f k8sops-deployment.yaml`                                                                                                            

```shell
namespace/kube-ops created
serviceaccount/k8s-operations created
clusterrole.rbac.authorization.k8s.io/k8s-operations created
clusterrolebinding.rbac.authorization.k8s.io/k8s-operations created
service/k8s-operations created
deployment.extensions/k8s-operations created
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzY4MzExNzcwLDEyNDk1MTQ0MDMsLTExMT
gyNDk1MzRdfQ==
-->
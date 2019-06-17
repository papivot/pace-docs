
# PKS K8S Cluster access

## Kubeconfig file format

For a user, a kubeconfig file is generally located in their #HOME/.kube folder. It is called `config` by default. The `kubectl` command-line tool uses kubeconfig files to find the information it needs to choose a cluster and communicate with the API server of a cluster. More details on kubeconfig can be found [here](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

A kubeconfig file is generally made up of four sections - clusters, context, users and current-context - 

```shell
apiVersion: v1
clusters:
users:
contexts:
current-context: 
```

##### Cluster 

The clusters section contains an array with information specific to the K8s clusters, namely the API server FQDN and port and the cluster root CA. 

```shell
clusters:
- cluster:
    certificate-authority-data: LS0tLS1...
    server: https://apicluster01.run.haas-254.pez.pivotal.io:8443
  name: cluster01
- cluster:
    certificate-authority-data: Ci0tLS0tQkV...
    server: https://cluster00.gcpcloud.navneetv.com:8443
  name: gcpcluster00
```

##### Users

The users section contains an array with information specific to the K8s user with access to cluster artifacts and associated auth information. This can vary depending on the auth provider. More details can be found [here](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/) - 
In a PKS environment, a sample user definition could be something similar to this. 

```shell
users:
- name: nverma
  user:
    auth-provider:
      config:
        client-id: pks_cluster_client
        cluster_client_secret: ""
        id-token: eyJhbG...
        idp-issuer-url: https://api.pks.gcpcloud.navneetv.com:8443/oauth/token
        refresh-token: eyJhbGciOi...
      name: oidc
```

##### Contexts
The contexts section contains an array of context associating the cluster and user objects from the above sections. An example can be similar to this - 

```shell
contexts:
- context:
    cluster: cluster01
    user: nverma
  name: cluster01
- context:
    cluster: gcpcluster00
    user: nverma
  name: gcpcluster00
```

##### Current-context

The current-context specifies which context is currently enforced and hence which cluster should the kubectl commands connect to and also which user to use for the connection to the cluster. 

```shell
current-context: gcpcluster00
```

Please study your config file.

## AuthN for Kubernetes 
 
 Some of the current and beta authentication mechanisms for Kubernetes are - 

 - [ ]  X509 Client Certs
 - [ ] Static Token File
 - [ ] Bootstrap Tokens
 - [ ] ~~Static Password File~~ - Disabled in PKS
 - [ ] Service Account Tokens
 - [ ] OpenID Connect Tokens
 - [ ] ~~Webhook Token~~
 - [ ] Authenticating Proxy
 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNzE3Mzc1NTgsLTM0ODUxMjg2MCw0Mj
IxNzc0Niw4NzQ3MzkwNTUsMTQ0ODMyMDg2NywxMjkwODAyOTA4
LC04NTE2MDU5NDddfQ==
-->
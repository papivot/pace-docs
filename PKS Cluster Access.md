
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
In a PKS environment, a sample user could be something similar to this. 

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

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTM4NTk3NDk0LC04NTE2MDU5NDddfQ==
-->
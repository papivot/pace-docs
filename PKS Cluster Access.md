
# PKS K8S Cluster access

## Kubeconfig file format

For a user, a kubeconfig file is generally located in their #HOME/.kube folder. It is called `config` by default. The `kubectl` command-line tool uses kubeconfig files to find the information it needs to choose a cluster and communicate with the API server of a cluster. More details on kubeconfig can be found [here](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

A kubeconfig file is generally made up of three sections - 

```shell
apiVersion: v1
clusters:
contexts:
current-context: gcpcluster00
kind: Config
preferences: {}
users:
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU4MDczNTEwMV19
-->
# Kubernetes fundamentals

## Kubectl fundamentals

To check is a valid kubeconfig file exists and kubectl get connect to the cluster using the config file ($HOME/.kube/config) - 
>`kubectl version`

If there are issues connecting with the K8s cluster, the output would be similar to this - 
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

else
it would be similar to this -
```shell
Client Version: version.Info{Major:"1", Minor:"14", GitVersion:"v1.14.3", GitCommit:"5e53fd6bc17c0dec8434817e69b04a25d8ae0ff0", GitTreeState:"clean", BuildDate:"2019-06-06T01:44:30Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.5", GitCommit:"2166946f41b36dea2c4626f90a77706f426cdea2", GitTreeState:"clean", BuildDate:"2019-03-25T15:19:22Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```

Some options on kubectl outputs - 

> `kubectl get nodes`

```shell
NAME                                   STATUS   ROLES    AGE   VERSION
045bbb78-a9cf-4087-b7d8-099f945f71ca   Ready    <none>   28d   v1.13.5
46be6e0b-127b-4338-9010-2cfe43e7cc38   Ready    <none>   28d   v1.13.5
...
```

>`kubectl get nodes -o wide`

```shell
NAME                                   STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
045bbb78-a9cf-4087-b7d8-099f945f71ca   Ready    <none>   28d   v1.13.5   172.28.0.5    172.28.0.5    Ubuntu 16.04.6 LTS   4.15.0-46-generic   docker://18.6.3
46be6e0b-127b-4338-9010-2cfe43e7cc38   Ready    <none>   28d   v1.13.5   172.28.0.6    172.28.0.6    Ubuntu 16.04.6 LTS   4.15.0-46-generic   docker://18.6.3
...
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAzNzM2NjczMSwxNjM1OTE4ODcwLDk5MT
k1NTQ1Nyw0OTY2MDczNzksLTE4NjMxNDc2MTMsNzMwOTk4MTE2
XX0=
-->
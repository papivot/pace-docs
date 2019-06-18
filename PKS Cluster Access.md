
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

### Cluster 

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

### Users

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

### Contexts
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
### Current-context

The current-context specifies which context is currently enforced and hence which cluster should the kubectl commands connect to and also which user to use for the connection to the cluster. 

```shell
current-context: gcpcluster00
```

Please study your config file.

## AuthN for Kubernetes 
 
 Details on K8S authentication can be found [here.](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) Some of the current and beta authentication mechanisms for Kubernetes (and thereby PKS) are - 

 - [ ]  X509 Client Certs
 - [ ] Static Token File
 - [ ] ~~Bootstrap Tokens~~ - Currently in Beta
 - [ ] ~~Static Password File~~ - Disabled in PKS by default
 - [ ] Service Account Tokens
 - [ ] OpenID Connect Tokens
 - [ ] ~~Webhook Token~~ - Disabled in PKS by default
 - [ ] Authenticating Proxy

### X509 Client Certs
The following configuration gets added to the K8S apiserver to allow using X509 Client cert files for authentication. 

```shell
--client-ca-file=/var/vcap/jobs/kube-apiserver/config/kubernetes-ca.pem 
```
##### Accessing cluster using X509 Client Certs
*Not a recommended method.* 

*Example [ This is for demonstration purpose only. This is specific to PKS and may vary for other K8s distributions] -* 

To achieve this, we need to generate client certificates using the root ca that stored on the K8S master server. 

This can be done by SSHing into the master using the BOSH CLI -

The following commands are executed from the OpsMan

> ```shell
> bosh alias-env gcp -e 10.0.0.5 --ca-cert /var/tempest/workspaces/default/root_ca_certificate
> bosh -e gcp log-in
> bosh -e gcp deployments
> bosh -e gcp -d service-instance_[instanceid_of_the_cluster] vms
> bosh -e gcp -d service-instance_[instanceid_of_the_cluster] ssh master/[master_vm_id]

Once logged in to the master vm execute the following - 
> ```shell
> cd /tmp
> openssl genrsa -out admin.key 4096
> openssl req -new -key admin.key -out admin.csr -subj "/CN=admin"
> openssl x509 -req -days 365 -sha256 -in admin.csr -CA /var/vcap/jobs/kube-controller-manager/config/cluster-signing-ca.pem -set_serial 2 -out admin.crt -CAkey /var/vcap/jobs/kube-controller-manager/config/cluster-signing-key.pem

This should create 3 files in the /tmp folder.

```shell
master/38ed427f-2998-4de0-8906-50e0accb8ae1:/tmp$ ls -1 admin*
admin.crt
admin.csr
admin.key
```
Copy the admin.crt and admin.key to the jumphost/bastion/desktop. Create a yaml file - admin.yaml - with the following content
```shell
# admin.yaml
piVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: temp-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: User
  name: admin
  namespace: default
```

With a kubeconfig that already has admin access, execute the following command to create a new clusterolebinding for user admin

>`kubectl apply -f admin.yaml`

This create a ClusterRoleBinding with cluster-admin role for the user named admin. 

You can now generate the kubeconfig using the project - [kubectl_plugin_login](https://github.com/papivot/kubectl_plugin_login)

- User is admin
- Keyfile is admin.key
- Certfile is admin.crt

##### Cleanup 

> `kubectl delete -f admin.yaml`
> `rm ~/.kube/config`

### Static Token File

The following configuration gets added to the K8S apiserver to allow using static token file for authentication. 

```shell
--token-auth-file=/var/vcap/jobs/kube-apiserver/config/tokens.csv
```
##### Accessing cluster using static token 
*Not a recommended method.* 

*Example [ This is for demonstration purpose only. This is specific to PKS and may vary for other K8s distributions] -* 

To achieve this, we need to get the admin token from one of the K8S master server. The location of the file is `/var/vcap/jobs/kube-apiserver/config/tokens.csv`

This can be done by SSHing into the master using the BOSH CLI -

The following commands are executed from the OpsMan

> ```shell
> bosh alias-env gcp -e 10.0.0.5 --ca-cert /var/tempest/workspaces/default/root_ca_certificate
> bosh -e gcp log-in
> bosh -e gcp deployments
> bosh -e gcp -d service-instance_[instanceid_of_the_cluster] vms
> bosh -e gcp -d service-instance_[instanceid_of_the_cluster] ssh master/[master_vm_id]
> cat /var/vcap/jobs/kube-apiserver/config/tokens.csv|grep admin
> ```

Once the Token has been retrieved, you can generate the kubeconfig using the project - [kubectl_plugin_login](https://github.com/papivot/kubectl_plugin_login)

 - User is admin
 - Token is the 1st value in the token.csv file's output. 

### Service Account Tokens

The following configuration gets added to the K8S apiserver to allow using Service Account Tokens for authentication.

```shell
--service-account-key-file=/var/vcap/jobs/kube-apiserver/config/service-account-public-key.pem
```
 This is the method to access clusters when **OIDC is not enabled** on PKS clusters.

##### Accessing cluster using Service Account Tokens 
*Not a highly recommended method.* 

*Example  [ This is for demonstration purpose only.] -* 

To achieve this, we need to get the token from one of the default service accounts and use this token to generate the Kubeconfig file.

> `kubectl get sa default -n default -o yaml`

This should output something similar to this. Note the name of the secrets.
```shell
ApiVersion: v1
kind: ServiceAccount
metadata:
 ....
 secrets:
- name: default-token-mcf5p
```
Use the secrets name to get the token -

> `kubectl get secrets default-token-mcf5p -n default  -o json |jq -r '.data.token'|base64 -D`

Once the Token has been retrieved, you can generate the kubeconfig using the project - [kubectl_plugin_login](https://github.com/papivot/kubectl_plugin_login)
 
 - User is default
 - Token is the base64 decoded value in the last step. 

**Note**: For regular use on non-OIDC enabled PKS clusters, once you are logged into PKS api, simply execute - 

> `pks get-credentials [pks_clustername]`

This creates the necessary service accounts, clusterroles (if needed), clusterrolebindings, secrets with tokens and the required kubeconfig file leveraging the token. 
 
### OpenID Connect Tokens

The following configuration gets added to the K8S apiserver when using UAA as OIDC provider. 

```shell
--oidc-ca-file=/var/vcap/jobs/kube-apiserver/config/oidc-ca.pem 
--oidc-client-id=pks_cluster_client 
--oidc-groups-claim=roles 
--oidc-groups-prefix= 
--oidc-issuer-url=https://api.pks.domain.com:8443/oauth/token 
--oidc-username-claim=user_name 
--oidc-username-prefix=- 
```

##### Accessing cluster using OIDC tokens 
*Highly recommended method.* 

[TO be expanded]

## PKS LDAP configuration

PKS LDAP integration is detailed [here.](https://community.pivotal.io/s/article/Configuring-LDAP-Integration-with-Pivotal-Cloud-Foundry) 

Sample configuration for LDAP integration for the PKS tile is provided here - 

 - [ ] **Server URL** `ldap[s]://ldap.domain.com:[389/636]`
 - [ ]  **LDAP Credentials** `uid=pksbind,ou=Users,o=5c73648e8c9a344401d08d84,dc=domain,dc=com`
 - [ ] **LDAP Password** `Pivotal123!`
 - [ ] **User Search Base** `ou=Users,o=5c73648e8c9a344401d08d84,dc=domain,dc=com`
 - [ ] **User Search Filter** `uid={0}`
 - [ ] **Group Search Base** `ou=Users,o=5c73648e8c9a344401d08d84,dc=jumpcloud,dc=com`
 - [ ] **Group Search Filter** `member={0}`
 - [ ] **First Name Attribute** `givenName`
 - [ ] **Last Name Attribute** `sn`
 - [ ] **Email Attribute** `mail`
 - [ ] **Email Domain(s)**
 - [ ] **LDAP Referrals** `“Automatically follow any referrals”`  
 - [ ] **External Groups Whitelist** `cluster_admin,cluster_dev,cluster_manager`

Once the LDAP has been configured and configurations applied, the relevant LDAP groups needs to be added to the PKS UAA groups pks.clusters.admin and pks.clusters.manage. This can be done by the following commands - 

> `
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI1NDQ2NywxOTQ3NjQ4NTE1LDgyNjAxMD
A1LDg3NDQyMzQ2NCwxMDQ0NTExOTMxLDExNTAxMjczMzIsLTEz
ODM5OTYxMzIsLTg0ODU0MDY2MiwtNTAxODU4NzA1LDEzODIwND
czNTIsLTk2MjcyODk1MywtMzU2MDQ5MDA0LDgxMjY4ODY2OCwt
MTQ1NzMzODM2NywtMTUxMjAxOTk4MiwxODMxNzY1MTAsNzc3OT
E5MjMxLC0xOTkyNzEwMTYwLC03NjQ1NjMxNjYsLTExNzE3Mzc1
NThdfQ==
-->
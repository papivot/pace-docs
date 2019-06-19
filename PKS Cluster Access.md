
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
apiVersion: rbac.authorization.k8s.io/v1
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

*Example [ This is for demonstration purpose only. This is specific to PKS and may vary for other K8s distributions] -* 

To provide OIDC token based authentication, the flag `Enable UAA as OIDC provider` has to be checked in the PKS tile in the UAA section. Doing so will configure created clusters as well as new clusters to use UAA as the OIDC provider. Once the setting has been enabled, the configuration has to be saved and changes applied. 

Once OIDC is enabled, a PKS admin can execute 
>`pks get-credentials [clustername]`

to get access to the cluster. During this process, besides generating the kubeconfig file, the CLI also creates a ClusterRoleBinding for the admin called `[username]-cluster-admin`

Create a yaml file - admin.yaml - with the following content
```shell
# admin.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pivotal-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: User
  name: appdev
  namespace: default
```
Execute the following command to create a new ClusterRoleBinding for user appdev

>`kubectl apply -f admin.yaml`

Once the necessary ClusterRoleBinding/RoleBinding has been created, you can generate the kubeconfig using the project - [kubectl_plugin_login](https://github.com/papivot/kubectl_plugin_login)

 - User is appdev
 - Token is the 1t value in the token.csv file's output. 


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

Once the LDAP has been configured and configurations applied, the relevant LDAP groups needs to be added to the PKS UAA groups **pks.clusters.admin** and **pks.clusters.manage**. This can be done by the following commands (after successfully login into the PKS UAA) - 

> ```shell
> uaac group map --name pks.clusters.admin cn=cluster_admin,ou=Users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com```

should return 

``` shell
Successfully mapped pks.clusters.admin to cn=cluster_admin,ou=Users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com for origin ldap
```

> ```shell
> uaac group map --name pks.clusters.manage cn=cluster_manager,ou=Users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com
> ```

should return 

```shell
Successfully mapped pks.clusters.manage to cn=cluster_manager,ou=Users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com for origin ldap
```
To validate the mapping has been successfully completed check the group mapping 

> `uaac group mappings`

should return similar to this - 

```shell
  resources
    ldap:
    -
      organizations.acme: cn=test_org,ou=people,o=springsource,o=org
    -
      pks.clusters.admin: cn=cluster_admin,ou=users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com
    -
      pks.clusters.manage: cn=cluster_manager,ou=users,o=5d09126b7cb23249d327bffe,dc=domain,dc=com
  schemas: urn:scim:schemas:core:1.0
  startindex: 1
  itemsperpage: 3
  totalresults: 3
```

To validate if user login is working use pks login to authenticate an LDAP user - 

> `pks login -k -a api.pks.domain.com -u [ldapuser] -p [password]`

should return 
```shell
API Endpoint: api.pks.domain.com
User: [ldapuser]
```

Once this is successful, an LDAP user entry [ldapuser] is created in the respective pks.clusters.[manage/admin] group within UAA.

> `uaac groups`

```shell
...
  pks.clusters.manage
    id: 96d579fc-63ab-49ca-ba0e-87784b2602f4
    meta
      version: 9
      created: 2019-06-04T22:51:30.000Z
      lastmodified: 2019-06-18T19:31:56.000Z
    description: Allows a user to manage PKS clusters
    members:
    -
      origin: ldap
      type: USER
      value: d8f65d06-824c-41ae-a751-12a47d21d762
    schemas: urn:scim:schemas:core:1.0
    zoneid: uaa
 ...
``` 

**NOTES:** 

 - Users and groups that are member of `pks.clusters.admin` and `pks.clusters.manage` can only access/authenticate to the PKS API endpoint. 
 - `pks.clusters.manage` group can only access the clusters that they create.
 - `pks get-kubeconfig` is only available when OIDC is enabled. 
 - Unless OIDC is enabled, all cluster authentication is thru service tokens. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA3ODA1NTM4NywtMzUxMjAyNjQwLC0yMT
QxNzExMjQxLC0xNTM4ODM3Njc3LC0xNjg5NzU2NTAyLC0yMDE2
NTMwNjE3LDEwNTQ0OTkyOTUsMTk0NzY0ODUxNSw4MjYwMTAwNS
w4NzQ0MjM0NjQsMTA0NDUxMTkzMSwxMTUwMTI3MzMyLC0xMzgz
OTk2MTMyLC04NDg1NDA2NjIsLTUwMTg1ODcwNSwxMzgyMDQ3Mz
UyLC05NjI3Mjg5NTMsLTM1NjA0OTAwNCw4MTI2ODg2NjgsLTE0
NTczMzgzNjddfQ==
-->
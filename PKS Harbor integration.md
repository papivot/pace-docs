
# PKS Harbor integration

### Requirements 
- Access to Pivotal Ops Manager UI with PKS tile deployed (*Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Upon a PKS environment has been created, you can login using the pks cli and create a cluster using an active plan.)*
- A working Harbor registry 
- A jumpbox with docker installed. 
- K8s clusters deployed in PKS (without and with UAA as OIDC provider)
- pks and kubectl binaries

## Configure Harbor
- Login to Harbor UI
- Create 3 users (Administration -> Users -> New User)
 - [ ] `devuser01`, `devuser01@domain.com`, `Dev User01`, `Passw0rd`
 - [ ] `devuser02`, `devuser02@domain.com`, `Dev User02`, `Passw0rd`
 - [ ] `projadmin01`, `projadmin01@domain.com`, `Project Admin01`, `Password`

- Create 3 projects 
 - [ ] `project-public-a` `Public`
 - [ ] `project-priv-a` `Private`, Assign `devuser01` role of `Developer` and `projadmin01` role of `Project Admin`
 - [ ] `project-priv-b` `Private`, Assign `devuser02` role of `Developer`

- Logout and login with each user and validate the scope and visibility of each user. 

## Upload Docker images to Harbor
- Login to the workstation that has Docker installed.
- Download the alpine container image from Dockerhub
> `docker pull alpine`

```shell
Using default tag: latest
latest: Pulling from library/alpine
921b31ab772b: Pull complete
Digest: sha256:ca1c944a4f8486a153024d9965aafbe24f5723c1d5c02f4964c045a16d19dc54
Status: Downloaded newer image for alpine:latest
```

> `docker images`

```shell
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
alpine                                                latest              4d90542f0623        3 weeks ago         5.58MB
```

- Tag the image appropriatly. 

> `docker tag alpine [Harbor_fqdn]/project-priv-a/alpine:v1`

> `docker tag alpine [Harbor_fqdn]/project-priv-b/alpine:v1`

> `docker images`

```shell
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
alpine                                                latest              4d90542f0623        3 weeks ago         5.58MB
[Harbor_fqdn]/project-priv-a/alpine   v1                  4d90542f0623        3 weeks ago         5.58MB
[Harbor_fqdn]/project-priv-b/alpine
```

- Login to the Harbor repository as `devuser01` and push both the images - 

> `docker login [Harbor_fqdn] -u devuser01`

If it gives an error - `x509: certificate signed by unknown authority` - this implies that docker does not trusts a self signed cert repository and an exception needs to be made for the Harbor registry. To do so, as root, create /modify the file `/etc/docker/daemon.json`  and add the following content -

```shell
{
  "insecure-registries" : ["harbor_fqdn"]
}
```
A service restart of docker daemon is required. Once completed, retry the login command once again. This time a successfully login message should be displayed.

```shell
Login Succeeded
```
>`docker push [Harbor_fqdn]/project-priv-a/alpine:v1`

should successfully complete. 

```shell
The push refers to repository [[Harbor_fqdn]/project-priv-a/alpine]
256a7af3acb1: Pushed
v1: digest: sha256:97a042bf09f1bf78c8cf3dcebef94614f2b95fa2f988a5c07314031bc2570c7a size: 528
```

>`docker push [Harbor_fqdn]/project-priv-b/alpine:v1`

should fail.

```shell
The push refers to repository [[Harbor_fqdn]/project-priv-b/alpine]
256a7af3acb1: Preparing
denied: requested access to the resource is denied
```

- Login as `devuser02` and push the image `alpine:v1`  to `project-priv-b`

> `docker login [Harbor_fqdn] -u devuser`

> `docker push [Harbor_fqdn]/project-priv-b/alpine:v1 `

## Use Harbor as a registry for K8S deployment 

We will now use the `alpine:v1` container image already uploaded to the Harbor registry to deploy it as a pod in a K8S cluster. 

Modify the below yaml with your [Harbor_fqdn] and save. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-no-auth
spec:
  containers:
  - image: [Harbor_fqdn]/project-priv-a/alpine:v1
    command:
      - /bin/sh
      - "-c"
      - "sleep 60m"
    imagePullPolicy: Always
    name: alpine-no-auth
  restartPolicy: Always
```

> `kubectl apply -f no-auth.yaml`


<!--stackedit_data:
eyJoaXN0b3J5IjpbNDk4NTQyMzM2LC0xOTA0NTM5MDk2LDEwNj
IyNDc1OTksMTYwMDgxMzEwNSwtMTU1MjQyMTkwMCwxODg4MTIz
MTExLDExOTczMzcxOTksLTQwNTczNzAzLC03NDEzODMyMzNdfQ
==
-->
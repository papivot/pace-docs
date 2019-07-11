
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

> `kubectl get pods -n default`

should return an error - 

```shell
NAME             READY   STATUS         RESTARTS   AGE
alpine-no-auth   0/1     ErrImagePull   0          69s
```

> `kubectl describe pod alpine-no-auth -n default`

should show the error message - 

```shell
...
vents:
  Type     Reason     Age                From                                              Message
  ----     ------     ----               ----                                              -------
  Normal   Scheduled  87s                default-scheduler                                 Successfully assigned default/alpine-no-auth to vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c
  Normal   Pulling    46s (x3 over 87s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  pulling image "harbor.pks.caracas.cf-app.com/project-priv-a/alpine:v1"
  Warning  Failed     46s (x3 over 86s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Failed to pull image "harbor.pks.caracas.cf-app.com/project-priv-a/alpine:v1": rpc error: code = Unknown desc = Error response from daemon: pull access denied for harbor.pks.caracas.cf-app.com/project-priv-a/alpine, repository does not exist or may require 'docker login'
  Warning  Failed     46s (x3 over 86s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ErrImagePull
  Normal   BackOff    6s (x6 over 86s)   kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Back-off pulling image "harbor.pks.caracas.cf-app.com/project-priv-a/alpine:v1"
  Warning  Failed     6s (x6 over 86s)   kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ImagePullBackOff

```

This happened because the Docker daemon running on the worker nodes does not have the required access to download the image and execute them. To do so, we need to provide the worker nodes with the required credentials to access the private registry. 

To do so, we first create a secret with the required credentials of `devuser01`

> `kubectl create secret docker-registry priv-a-creds --docker-server=[Harbor_fqdn] --docker-username=devuser01 --docker-password=Passw0rd --docker-email=devuser01@domain.com -n default`

This creates the required secret-

```
secret/priv-a-creds created
```

To view the secret, execute the following command- 

> `kubectl get secret/priv-a-creds -o json |jq -r .data.\".dockerconfigjson\"|base64 --decode;echo`

should display something like this -
```shell
{"auths":{"[Harbor_fqdn]":{"username":"devuser01","password":"Passw0rd","email":"devuser01@domain.com","auth":"ZGV2dXNlcjAxOlBhc3N3MHJk"}}}
```

Leveraging the secret, deploy a new yaml file (modifying the [Harbor_fqdn] value)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-auth
spec:
  containers:
  - image: [Harbor_fqdn]/project-priv-a/alpine:v1
    command:
      - /bin/sh
      - "-c"
      - "sleep 60m"
    imagePullPolicy: Always
    name: alpine-auth
  restartPolicy: Always
  imagePullSecrets:
  - name: priv-a-creds
```

> `kubectl apply -f auth.yaml`

> `kubectl get pods -n default`

should show a successfully running pod -

```shell
NAME             READY   STATUS             RESTARTS   AGE
alpine-auth      1/1     Running            0          8s
alpine-no-auth   0/1     ImagePullBackOff   0          54m
```

#### Cleanup

> `kubeclt delete -f no-auth.yaml`

> `kubectl delete -f auth.yaml`


## Harbor Clair 

We will look at Harbor's vulnerability scanning and management capabilities in this section. 

- Within Harbor UI->Administration-> Configuration-> Vulnerability, make sure that  `Database updated on` has a valid time.

- Download an old Docker container, tag it and push it to Harbor -

> `docker image pull centos:centos7.2.1511`

> ` docker tag centos:centos7.2.1511 [Harbor_fqdn]/project-priv-a/centos7:v1`

> `docker push [Harbor_fqdn]/project-priv-a/centos7:v1`

Check the repository in Harbor UI and confirm that the v1 image has been uploaded.

- Now, we will update the docker container, create a new image and upload it to Harbor. 

> `docker run -t -d [Harbor_fqdn]/project-priv-a/centos7:v1`

> `docker ps`

This should return the ID of a running container. Grab that ID use it in the next command - 

> `docker exec -it cadf8ff0f00f bash`

where `cadf8ff0f00f` was the ID of the running centos7:v1 container. Within the container, update all the rpm binaries.

> `yum udpate -y`

Once done `exit` out of the container and grab the container ID.

> `exit`

> ` docker ps`

> `docker commit cadf8ff0f00f [Harbor_fqdn]/project-priv-a/centos7:v2`

where `cadf8ff0f00f` is the ContainerID of the centos7 container. 

Push the newly created image to Harbor as v2.

> `docker push [Harbor_fqdn]/project-priv-a/centos7:v2`

- Within the Harbor UI navigate to Projects -> project-priv-a -> centos7. There should be two images v1 and v2. Select both and scan them. Once the scan is completed, observe the results. v1 should have a number of vulnerabilities while v2 should be clean. 

Navigate to Projects -> project-priv-a-> Configuration and set the following - 

`Automatically scan images on push`
`Prevent vulnerable images from running.`
`Prevent images with vulnerability severity of Medium and above from being deployed.`

Save. 

Now try to execute the v1 pod, that has a number of medium and high vulnerabilities. 

> `kubectl run --generator=run-pod/v1 riskypod --image=[Harbor_fqdn]/project-priv-a/centos7:v1 -n default`

> `kubectl describe pod riskypod -n default`

This should show that the pod was unable to be scheduled - 

```shell
...
Events:
  Type     Reason     Age                From                                              Message
  ----     ------     ----               ----                                              -------
  Normal   Scheduled  33s                default-scheduler                                 Successfully assigned default/riskypod to vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c
  Normal   Pulling    21s (x2 over 32s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  pulling image "harbor.pks.caracas.cf-app.com/project-priv-a/centos7:v1"
  Warning  Failed     21s (x2 over 32s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Failed to pull image "harbor.pks.caracas.cf-app.com/project-priv-a/centos7:v1": rpc error: code = Unknown desc = Error response from daemon: unknown: The severity of vulnerability of the image: "high" is equal or higher than the threshold in project setting: "medium".
  Warning  Failed     21s (x2 over 32s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ErrImagePull
  Normal   BackOff    7s (x2 over 32s)   kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Back-off pulling image "harbor.pks.caracas.cf-app.com/project-priv-a/centos7:v1"
  Warning  Failed     7s (x2 over 32s)   kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ImagePullBackOff
```

## Notary

- Within the Harbor UI, navigate to Projects -> project-public-a-> Configuration and set the following
`Enable content trust`
`Save`.
 Within the Harbor UI, navigate to Projects -> project-public-a->Repositories and download the `Registry Certificate`
- On the Linux jumpbox, that has the Docker daemn running copy the downloaded certificate. 
> `mkdir ~/.docker/tls/[Harbor_fqdn]\:4443`

> `cp ca.crt ~/.docker/tls/[Harbor_fqdn]\:4443`

- Download an alpine container from Dockerhub and upload v1 without trust enabled. 
> `docker pull alpine`

> `docker login harbor.pks.caracas.cf-app.com -u admin`

```shell
Login Succeeded
```
> `docker tag alpine [Harbor_fqdn]/project-public-a/alpine:v1`

> `docker push [Harbor_fqdn]/project-public-a/alpine:v1`

- Modify the apline:v1 docker image by running a package update on it. After that we will enabled Trust and upload the image in Harbor as v2.

> `docker run -t -d [Harbor_fqdn]/project-public-a/alpine:v1`
> `docker ps`
> `docker exec -it 3655d9703e56 sh`

where 3655d9703e56 is the container id of the docker ps command output for the Alpine container. 

> `apk update && apk upgrade && apk add curl && rm -rf /var/cache/apk/*`
> `exit`

> `docker ps`
> `docker commit 3655d9703e56 [Harbor_fqdn]/project-public-a/alpine:v2`

where 3655d9703e56 is the container id of the docker ps command output for the Alpine container.

> `export DOCKER_CONTENT_TRUST_SERVER=https://[Harbor_fqdn]:4443`
> `export DOCKER_CONTENT_TRUST=1`

Now when we push the image to the Harbor registry, the image will be signed and uploaded. This can be validated in the Harbor UI. 

> `docker push [Harbor_fqdn]/project-public-a/alpine:v2`

```shell
The push refers to repository [[Harbor_fqdn]/project-public-a/alpine]
d9f8178ae34c: Layer already exists
256a7af3acb1: Layer already exists
v2: digest: sha256:c685267c7f51fd5516a0d3be1f93a4fde782f4a17bcc6a4cfead2b1f6056b009 size: 738
Signing and pushing trust metadata
Enter passphrase for root key with ID e107c01:
Enter passphrase for new repository key with ID e6331d5:
Repeat passphrase for new repository key with ID e6331d5:
Finished initializing "[Harbor_fqdn]/project-public-a/alpine"
Successfully signed [Harbor_fqdn]/project-public-a/alpine:v2
```

> `docker trust inspect --pretty [Harbor_fqdn]/project-public-a/alpine`

```shell

Signatures for [Harbor_fqdn]/project-public-a/alpine

SIGNED TAG          DIGEST                                                             SIGNERS
v2                  c685267c7f51fd5516a0d3be1f93a4fde782f4a17bcc6a4cfead2b1f6056b009   (Repo Admin)

Administrative keys for [Harbor_fqdn]/project-public-a/alpine

  Repository Key:	e6331d505e0a00eeeab92d6974190aca47b99098fd7c6802fadd2098cf741414
  Root Key:	2180e7f440668868965edfe646359c22a2b3c58298ec43e0e8e530a90ac3e872
```

- We will now execute the alpine:v2 pod within a K8s cluster using the following yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-signed-pod
spec:
  containers:
  - image: [Harbor_fqdn]/project-public-a/alpine:v2
    command:
      - /bin/sh
      - "-c"
      - "sleep 60m"
    imagePullPolicy: Always
    name: alpine-signed-pod
  restartPolicy: Always
```

> `kubectl apply -f signed.yaml`

> `kubectl get pods -n default`

should show an `alpine-signed-pod` running
```shell
NAME                READY   STATUS             RESTARTS   AGE
alpine-signed-pod   1/1     Running            0          6s
```

- We will now revoke the trust of the image and then try to schedule it on the K8s cluster. 

> `kubectl delete pod alpine-signed-pod -n default`

> `docker trust revoke  [Harbor_fqdn]/project-public-a/alpine:v2`

```shell
Enter passphrase for repository key with ID e6331d5:
Successfully deleted signature for [Harbor_fqdn]/project-public-a/alpine:v2
```
> `docker trust inspect --pretty [Harbor_fqdn]/project-public-a/alpine`

```shell
No signatures for [Harbor_fqdn]/project-public-a/alpine

Administrative keys for [Harbor_fqdn]/project-public-a/alpine

  Repository Key:	e6331d505e0a00eeeab92d6974190aca47b99098fd7c6802fadd2098cf741414
  Root Key:	2180e7f440668868965edfe646359c22a2b3c58298ec43e0e8e530a90ac3e872
```

Harbor UI should now show that alpine:v2 image is no longer signed. 
>`kubectl apply -f signed.yaml`
> `kubectl describe pods alpine-signed-pod`

```shell
...
Events:
  Type     Reason     Age               From                                              Message
  ----     ------     ----              ----                                              -------
  Normal   Scheduled  22s               default-scheduler                                 Successfully assigned default/alpine-signed-pod to vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c
  Normal   BackOff    20s               kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Back-off pulling image "[Harbor_fqdn]/project-public-a/alpine:v2"
  Warning  Failed     20s               kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ImagePullBackOff
  Normal   Pulling    6s (x2 over 21s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  pulling image "[Harbor_fqdn]/project-public-a/alpine:v2"
  Warning  Failed     6s (x2 over 21s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Failed to pull image "[Harbor_fqdn]/project-public-a/alpine:v2": rpc error: code = Unknown desc = Error response from daemon: unknown: The image is not signed in Notary.
  Warning  Failed     6s (x2 over 21s)  kubelet, vm-598b94c9-cf6b-4adc-45be-29d02a22ac9c  Error: ErrImagePull
```

#### Cleanup

kubectl delete -f signed.yaml
kubeclt delete -f auth.yaml
kubectl delete -f no-auth.yaml
kubectl delete pods riskypod -n default
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc0OTc0MDc2LC0xOTQ2MjgzNjAxLC04NT
I2ODIzNTgsMjAwMjAyMDI2MCwxODk3NTgwMjU5LC0xNjIzMDcy
MzI2LDE3MzU0MDY3MjgsLTk4OTk3NTgxNiw4MTk5NzA4MjEsMT
g5NjQ1OTUwMCwtMTMyMzc1NjE2LDQ5ODU0MjMzNiwtMTkwNDUz
OTA5NiwxMDYyMjQ3NTk5LDE2MDA4MTMxMDUsLTE1NTI0MjE5MD
AsMTg4ODEyMzExMSwxMTk3MzM3MTk5LC00MDU3MzcwMywtNzQx
MzgzMjMzXX0=
-->
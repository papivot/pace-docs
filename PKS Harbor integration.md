
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

## Upload Docker images to harbor
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

> `docker tag alpine harbor.pks.caracas.cf-app.com/project-priv-a/alpine:v1`

> `docker tag alpine harbor.pks.caracas.cf-app.com/project-priv-b/alpine:v1`

> `docker images`

```shell
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
alpine                                                latest              4d90542f0623        3 weeks ago         5.58MB
harbor.pks.caracas.cf-app.com/project-priv-a/alpine   v1                  4d90542f0623        3 weeks ago         5.58MB
harbor.pks.caracas.cf-app.com/project-priv-b/alpine
```

- Login to the Harbor repository as `devuser01` and push both the images - 

> 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NTI0MjE5MDAsMTg4ODEyMzExMSwxMT
k3MzM3MTk5LC00MDU3MzcwMywtNzQxMzgzMjMzXX0=
-->
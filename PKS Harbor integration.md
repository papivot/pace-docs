
# PKS Harbor integration

### Requirements 
- Access to Pivotal Ops Manager UI with PKS tile deployed (*Pivotal employees connected thru the VPN can request a PKS environment in GCP thru [Toolsmith](https://environments.toolsmiths.cf-app.com/home). Upon a PKS environment has been created, you can login using the pks cli and create a cluster using an active plan.)*
- A working Harbor registry 
- A jumpbox with docker installed. 
- K8s clusters deployed in PKS (without and with UAA as OIDC provider)
- pks and kubectl binaries

## Configure Harbor
- Login to Harbor UI
- Create 3 projects 
 - [ ] `project-public-a` `Public`
 - [ ] `project-priv-a` `Private`
 - [ ] `project

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4NTkyMTAzOCwtNzQxMzgzMjMzXX0=
-->
# PKS Backup and Restore <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [References](#references)
- [Requirements](#requirements)
- [Configure Jumpbox](#configure-jumpbox)
- [Backup Installation Settings](#backup-installation-settings)
- [Backup PKS BOSH Director](#backup-pks-bosh-director)
- [Restore PKS BOSH Director](#restore-pks-bosh-director)
- [Backup PKS Control Plane](#backup-pks-control-plane)
  - [Find the Deployment Name](#find-the-deployment-name)
  - [Run the Pre-Backup Checks](#run-the-pre-backup-checks)
- [Restore PKS Control Plane](#restore-pks-control-plane)
- [Backup PKS Cluster Deployments](#backup-pks-cluster-deployments)

## References
  * https://docs.pivotal.io/runtimes/pks/1-4/bbr-backup.html
  * https://docs.pivotal.io/runtimes/pks/1-4/bbr-restore.html

## Requirements 
- Access to Pivotal Ops Manager UI with PKS tile deployed
- SSH Access to Pivotal Ops Manager
- A K8s cluster deployed in PKS
- Access to the above K8s cluster
- Access to the IaaS running the PKS environment
- Kubectl binary
- BBR binary
- OM binary

## Configure Jumpbox

TODO
```
export $(om -k --target https://pcf.yourdomain.com -u $OPSMAN_USER -p $OPSMAN_PASS curl --path /api/v0/deployed/director/credentials/bosh_commandline_credentials | jq -r .credential)
get bosh creds
install bbr binary
install om binary
get bosh and bbr certs
bosh alias-env
bosh login
```

## Backup Installation Settings

```
om -k -t https://opsman.yourdomain.com -u $OPSMAN_USER -p $OPSMAN_PASS export-installation -o installation.zip
```

Validate backup:

```
$ unzip -l installation.zip 
Archive:  installation.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
    63502  2019-07-08 19:09   uaa_database_dump.postgres
        0  2019-07-08 19:09   deployments/
     2655  2019-07-08 19:09   deployments/bosh-state.json
   106675  2019-07-08 19:09   installation.yml
   693376  2019-07-08 19:09   rails_database_dump.postgres
        0  2019-07-08 19:09   metadata/
   282662  2019-07-08 19:09   metadata/pivotal-container-service-1.4.1-build.4.yml
---------                     -------
  1148870                     7 files
```

## Backup PKS BOSH Director

```
$ bbr director --host $BOSH_ENVIRONMENT --username bbr --private-key-path bbr.pem backup
[bbr] 2019/07/08 17:58:21 INFO - Looking for scripts
[bbr] 2019/07/08 17:58:21 INFO - bosh/0/bbr-credhubdb/backup
[bbr] 2019/07/08 17:58:21 INFO - bosh/0/bbr-credhubdb/restore
[bbr] 2019/07/08 17:58:21 INFO - bosh/0/blobstore/backup
...
[bbr] 2019/07/08 18:02:54 INFO - Finished validity checks -- for job blobstore on bosh/0...
[bbr] 2019/07/08 18:02:54 INFO - Backup created of 10.0.0.10 on 2019-07-08 17:58:25.569419494 +0000 UTC m=+3.828220386
[bbr] 2019/07/08 18:02:54 INFO - Cleaning up...
```

At this point you will have a backup directory in the the current working directory:

```
$ ls -l
total 12
drwx------ 2 user user 4096 Jul  8 17:58 10.0.0.10_20190708T175821Z
...
```

**Note:** It's always best practice to:
  * Move the backup artifacts off the jumpbox / opsman VM to permanent storage space
  * Compress and encrypt the backup artifacts when storing them
  * Make redundant copies of your backup and store them in multiple locations

## Restore PKS BOSH Director

TODO

## Backup PKS Control Plane

### Find the Deployment Name

```
$ export DEPLOYMENT_NAME=$(bosh -e gcp deployments | grep ^pivotal-container-service | awk '{print $1}')
$ echo $DEPLOYMENT_NAME
pivotal-container-service-178f901c363756e15a06
```

Review the returned output. The PKS BOSH deployment name begins with `pivotal-container-service` and includes a unique identifier. In the example output above, the BOSH deployment name is `pivotal-container-service-178f901c363756e15a06`.

### Run the Pre-Backup Checks

```
$ bbr deployment -t $BOSH_ENVIRONMENT -u $BOSH_CLIENT -p $BOSH_CLIENT_SECRET -d $DEPLOYMENT_NAME --ca-cert root.pem pre-backup-check

[bbr] 2019/07/09 17:35:05 INFO - Looking for scripts
...
[bbr] 2019/07/09 17:35:09 INFO - Running pre-checks for backup of pivotal-container-service-178f901c363756e15a06...
[17:35:09] Deployment 'pivotal-container-service-178f901c363756e15a06' can be backed up.
 ```

### Back Up the Control Plane

```
$ bbr deployment -t $BOSH_ENVIRONMENT -u $BOSH_CLIENT -d $DEPLOYMENT_NAME --ca-cert root.pem backup --with-manifest

[bbr] 2019/07/09 17:49:49 INFO - Looking for scripts
...
[bbr] 2019/07/09 17:49:52 INFO - Running pre-checks for backup of pivotal-container-service-178f901c363756e15a06...
[bbr] 2019/07/09 17:49:52 INFO - Starting backup of pivotal-container-service-178f901c363756e15a06...
[bbr] 2019/07/09 17:49:52 INFO - Running pre-backup-lock scripts...
...
[bbr] 2019/07/09 17:49:58 INFO - Running backup scripts...
...
[bbr] 2019/07/09 17:49:59 INFO - Finished backing up pks-api on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597.
[bbr] 2019/07/09 17:49:59 INFO - Finished backing up bbr-uaadb on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597.
[bbr] 2019/07/09 17:49:59 INFO - Finished running backup scripts.
[bbr] 2019/07/09 17:49:59 INFO - Running post-backup-unlock scripts...
...
[bbr] 2019/07/09 17:53:34 INFO - Finished running post-backup-unlock scripts.
[bbr] 2019/07/09 17:53:34 INFO - Copying backup -- 16K uncompressed -- for job pks-api on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:34 INFO - Copying backup -- 52K uncompressed -- for job bbr-uaadb on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:34 INFO - Finished copying backup -- for job bbr-uaadb on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:34 INFO - Starting validity checks -- for job bbr-uaadb on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:34 INFO - Finished copying backup -- for job pks-api on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:34 INFO - Starting validity checks -- for job pks-api on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:35 INFO - Finished validity checks -- for job bbr-uaadb on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:35 INFO - Finished validity checks -- for job pks-api on pivotal-container-service/7f9d9701-85f2-4763-9bad-682853859597...
[bbr] 2019/07/09 17:53:35 INFO - Backup created of pivotal-container-service-178f901c363756e15a06 on 2019-07-09 17:53:34.167266436 +0000 UTC m=+224.489611431
```

Check that the backup directory was successfully created:
```
$ ls -l | grep pivotal-container-service
drwx------ 2 user user   4096 Jul  9 17:53 pivotal-container-service-178f901c363756e15a06_20190709T174949Z
```

## Restore PKS Control Plane

TODO

## Backup PKS Cluster Deployments

TODO
# Docker fundamentals

## Docker installation

**Note**: Most of the demos in this section are based on Ubuntu 18.4 running as an t2.medium EC2 instance. Please modify your environment accordingly.

#### Step1 - Update apt
`sudo apt-get update`

#### Step 2 - Install Docker packages
`sudo apt install docker.io`

#### Step 3 - Enable and start Docker service
`sudo systemctl start docker`
`sudo systemctl enable docker`

#### Step 4 - Verify Docker service is running
`ps -eaf|grep docker`

should return something similar to - 
```
root      4234     1  0 16:19 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

## Executing your first container

#### Step 1 - Run Hello-world
`sudo docker run hello-world`

should return something similar to this - 
```Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
... 
```
#### Step 2 - View running containers
`sudo docker ps -a`

should return something similar to this - 
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
7bbe9e08ddef        hello-world         "/hello"            4 minutes ago       Exited (0) 4 minutes ago                       pensive_thompson
```

#### Step 3 - View local image repository 
`sudo docker images `

should return something similar to this - 
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
```

#### Step 4 - Cleanup

Use the **CONTAINER ID** captured in Step 2, to remove the container - 

` 
sudo docker rm 7bbe9e08ddef
`

Use the **IMAGE ID** to remove the image from the local repository - 

` 
sudo docker rmi fce289e99eb9
`

Should return something like this -
```
Untagged: hello-world:latest
Untagged: hello-world@sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
```

## Building your first container 

#### Download the sample code from Github 

Download the sample code from the repository provided in the example. This sample code will be used multiple times during the future demos. 

```shell
$> cd workshop
$> git clone https://github.com/papivot/k8s-operations.git
Cloning into 'k8s-operations'...
Username for 'https://github.com': nverma@pivotal.io
Password for 'https://nverma@pivotal.io@github.com':
remote: Enumerating objects: 29, done.
remote: Counting objects: 100% (29/29), done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 29 (delta 15), reused 7 (delta 5), pack-reused 0
Unpacking objects: 100% (29/29), done.
```
```shell
$> cd k8s-operations/
$> curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$> chmod +x kubectl
```

Study the Dockerfile included on the k8s-operations folder and the syntax for the various commands. Additional details can be found in [Dockerfile Reference Page](https://docs.docker.com/engine/reference/builder/)


<!--stackedit_data:
eyJoaXN0b3J5IjpbODczNzM3ODQyLC05MzM4NDEyOTAsLTEwMD
EzMzMwMDMsODUxMzEzOTAxLDEwNjk3MDcyMzEsMzc0MjU2MTg3
LC0xODI5NjYyNDU3XX0=
-->
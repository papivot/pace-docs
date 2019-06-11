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
`root      4234     1  0 16:19 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock`

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
... ```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...

#### Step 2 - Check 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwMzU0ODYxNTQsMzc0MjU2MTg3LC0xOD
I5NjYyNDU3XX0=
-->
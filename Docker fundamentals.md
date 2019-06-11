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

## Running first 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU1NDE5NzgzNSwzNzQyNTYxODcsLTE4Mj
k2NjI0NTddfQ==
-->
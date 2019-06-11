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
```
Unable to find image 'hello-world:latest' locally
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

#### Step 1 - Download the sample code from Github 

Download the sample code from the repository provided in the example. You may have to provide the Github login credentials, if the repository is marked private. This sample code will be used multiple times during the future demos. 

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

Download the **kubectl** binary as its required by this specific container image. 

```shell
$> cd k8s-operations/
$> curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$> chmod +x kubectl
```

Study the Dockerfile included on the k8s-operations folder and the syntax for the various commands. Additional details can be found in [Dockerfile Reference Page](https://docs.docker.com/engine/reference/builder/)

#### Step 2 - Build the Docker image. 

While in the k8s-operations folder, run the following command to build the Docker image locally. This will save the image in the local repository with the tag of k8soper:0.0.1

`sudo docker build -t k8soper:0.0.1 .`

The output would be similar to this - 

```
 ---> Running in cb4d59a8e413
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
v3.9.4-24-g4e2ff29bbe [http://dl-cdn.alpinelinux.org/alpine/v3.9/main]
v3.9.4-26-ga3169d5242 [http://dl-cdn.alpinelinux.org/alpine/v3.9/community]
OK: 9770 distinct packages available
(1/11) Installing libbz2 (1.0.6-r6)
(2/11) Installing expat (2.2.6-r0)
(3/11) Installing libffi (3.2.1-r6)
(4/11) Installing gdbm (1.13-r1)
(5/11) Installing xz-libs (5.2.4-r0)
(6/11) Installing ncurses-terminfo-base (6.1_p20190105-r0)
(7/11) Installing ncurses-terminfo (6.1_p20190105-r0)
(8/11) Installing ncurses-libs (6.1_p20190105-r0)
(9/11) Installing readline (7.0.003-r1)
(10/11) Installing sqlite-libs (3.28.0-r0)
(11/11) Installing python3 (3.6.8-r2)
Executing busybox-1.29.3-r10.trigger
OK: 68 MiB in 25 packages
Collecting schedule
  Downloading https://files.pythonhosted.org/packages/57/22/3a709462eb02412bd1145f6e53604f36bba191e3e4e397bea4a718fec38c/schedule-0.6.0-py2.py3-none-any.whl
Installing collected packages: schedule
Successfully installed schedule-0.6.0
You are using pip version 18.1, however version 19.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container cb4d59a8e413
 ---> c95f096b985b
Step 3/15 : RUN mkdir -p /usr/local/bin && mkdir -p /user/k8soper
 ---> Running in 09e4033d644b
Removing intermediate container 09e4033d644b
 ---> c553a156cd9b
Step 4/15 : ENV HOME=/user/k8soper
 ---> Running in bc6412bccd70
Removing intermediate container bc6412bccd70
 ---> ef11160fcd57
Step 5/15 : ADD ./kubectl /usr/local/bin/kubectl
 ---> 0f709840e7bd
Step 6/15 : ADD ./dumpconfig.sh /usr/local/bin/dumpconfig.sh
 ---> 389186e81be2
Step 7/15 : ADD ./exportjson.py /usr/local/bin/exportjson.py
 ---> b3b56a68ebf1
Step 8/15 : ADD ./dockerrun.sh /usr/local/bin/dockerrun.sh
 ---> 42124b3d5ee4
Step 9/15 : ADD ./runhttp.py /usr/local/bin/runhttp.py
 ---> a916d709d0a7
Step 10/15 : RUN chmod +x /usr/local/bin/kubectl 	&& chmod +x /usr/local/bin/dumpconfig.sh 	&& chmod +x /usr/local/bin/runhttp.py 	&& chmod +x /usr/local/bin/exportjson.py 	&& chmod +x /usr/local/bin/dockerrun.sh
 ---> Running in 2fa2f43cf897
Removing intermediate container 2fa2f43cf897
 ---> b725ade0f54d
Step 11/15 : RUN adduser k8soper -Du 9999 -h /user/k8soper
 ---> Running in 15dbe8385e2a
Removing intermediate container 15dbe8385e2a
 ---> 9d6ec104e074
Step 12/15 : USER k8soper
 ---> Running in 2620fea722a7
Removing intermediate container 2620fea722a7
 ---> 0c82b851cc94
Step 13/15 : WORKDIR /user/k8soper
 ---> Running in 35b664b7c179
Removing intermediate container 35b664b7c179
 ---> a9de2932dcfc
Step 14/15 : EXPOSE 8080
 ---> Running in 6608779cb862
Removing intermediate container 6608779cb862
 ---> d55e90bf05ab
Step 15/15 : CMD ["/usr/local/bin/dockerrun.sh"]
 ---> Running in 6b058c3e884a
Removing intermediate container 6b058c3e884a
 ---> 414d93beb557
Successfully built 414d93beb557
Successfully tagged k8soper:0.0.1
```
Look at the above output and try to understand how each operation relates to instructions provided in the Dockerfile. In this example, the final output is an image with an ID of **414d93beb557** and a tag of **k8soper:0.0.1**. This can be verified by the following command - 

`sudo docker images`

Output would be similar to 
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
k8soper             0.0.1               414d93beb557        19 minutes ago      144MB
alpine              latest              055936d39205        4 weeks ago         5.53MB
```

#### Step 3 - Push the image to external repository. 

**Note** This process may differ for different external registries. While the process is similar, whereby the Docker daemon needs to authenticate to the external registry first and then push the image to the external registry. The authentication process may vary for each registry. We will be pushing the image to GCR in this example. Pushing images to Harbor will be discussed later in the workshop. 

 - [ ] Authenticate to GCR

`sudo gcloud auth configure-docker`

```shell
The following settings will be added to your Docker config file
located at [/home/ubuntu/.docker/config.json]:
 {
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  Y

Docker configuration file updated.
```
Execute the following command to configure gcloud to set the account settings and credentials - 

`gcloud auth login`

```shell
Go to the following link in your browser:

    https://accounts.google.com/o/oauth2/auth?redirect_uri=.....

Enter verification code: 4/ZgEwtiYo.........
WARNING: `gcloud auth login` no longer writes application default credentials.
If you need to use ADC, see:
  gcloud auth application-default --help

You are now logged in as [username@domain.com].
```

 - [ ] Tag and push image

Once you have authenticated to the registry, it is time to push the image. These steps should be similar for all registries. Grab the image ID that you want to push. In this case the value is - **414d93beb557**

`sudo docker images`

```shell
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
k8soper             0.0.1               414d93beb557        About an hour ago   144MB
alpine              latest              055936d39205        4 weeks ago         5.53MB
```

Tag the image with the appropriate registry specific tags. For GCR we will use the format gcr.io / [username] - 

`sudo docker tag 414d93beb557 gcr.io/[username]/k8soper:0.0.1`

Make sure that the tag went into effect - 

`sudo docker images`

```shell
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
k8soper                    0.0.1               414d93beb557        About an hour ago   144MB
gcr.io/pa-nverma/k8soper   0.0.1               414d93beb557        About an hour ago   144MB
alpine                     latest              055936d39205        4 weeks ago         5.53MB
```

Now push the image - 

`sudo docker push gcr.io/[username]/k8soper:0.0.1`

This should give an output similar to - 

```shell
The push refers to repository [gcr.io/pa-nverma/k8soper]
c02522c797e1: Pushed
382b23168923: Pushed
97cfbf18e0ca: Pushed
96b0e7a6d0d9: Pushed
1f6e4ee237d7: Pushed
ecf96d8d39cf: Pushed
f8ed2a793d04: Pushed
ac169c9860eb: Pushed
d5f9e6c07599: Pushed
f1b5933fe4b5: Layer already exists
0.0.1: digest: sha256:3d76e9859e5559b62a5d8e16a600c2f418d85370118ae9e382bef57d01a4328e size: 2408
```

Check the registry and verify that the image has been uploaded. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgzNzk1NjM1MywtMjA3MzIwMjI3NCwtOD
M3OTU2MzUzLC0xOTA1NDIzMzM2LDQwMDQ1MzczMywxNzI4OTY1
OTUzLDE3MzI0MjQ0MjcsMjA3NjA3NjUwMywtMjA2NjcyOTc4LD
EyMTk5NzY0ODcsODczNzM3ODQyLC05MzM4NDEyOTAsLTEwMDEz
MzMwMDMsODUxMzEzOTAxLDEwNjk3MDcyMzEsMzc0MjU2MTg3LC
0xODI5NjYyNDU3XX0=
-->
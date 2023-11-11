# Securing the Docker Daemon
Regular port: 2375

TLS port: 2376

## 1. Secure communication
- **We need to encrypt the communication between the Docker client and the server**

On the server:
- Create a keypair using openssl. You can use [this script by Alexis Ahmed](https://github.com/AlexisAhmed/DockerSecurityEssentials/blob/main/Docker-TLS-Authentication/secure-docker-daemon.sh).
- Create a config file that sepcifies how the docker daemon to run on startup
  ```
  sudo mkdir /etc/systemd/system/docker.service.d
  sudo vim /etc/systemd/system/docker.service.d/override.conf

  # Add the following contents in the conf file
  [Service]
  ExecStart=
  ExecStart=/usr/bin/dockerd -D -H unix:///var/run/docker.sock --tlsverify --tlscert=/home/user/.docker/server-cert.pem --tlscacert=/home/user/.docker/ca.pem --tlskey=/home/user/.docker/server-key.pem -H tcp://0.0.0.0:2376

  # Restart daemon and service
  sudo systemctl daemon-reload
  sudo systemctl restart docker
  sudo systemctl status docker  

  netstat -anp | grep 2376
  ```

On the client:
- Copy these files to the client
  ```
  key.pem
  cert.pem
  ca.pem
  ```
- Configure the following environment variables
  ```
  export DOCKER_TLS_VERIFY="1"
  export DOCKER_HOST_IP="tcp://SERVER_IP:2376"
  export DOCKER_CERT_PATH="/home/clientuser/.docker/"
  ```

## 2. User Namespaces
- Namespaces are a feature of the Linux kernel that provide isolation of kernel resources. One process sees one set of resources, another process sees another set of resources.
- Namespaces are used to isolate resources like users, networks, processIDs among different containers.
- Running a docker container with the default namespace will make it run with the root account - this is insecure.
- **We need to reconfigure the docker daemon to make it use a non-default namespace, i.e. a user name space.**
  - We can use the `dockremap` user that is provided by Docker.
  - This is done by adding an extra argument `--userns-remap="default"` in the above config file
    ```
    sudo vim /etc/systemd/system/docker.service.d/override.conf

    # Add the following content in the conf file
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd -D -H unix:///var/run/docker.sock --tlsverify --tlscert=/home/user/.docker/server-cert.pem --tlscacert=/home/user/.docker/ca.pem --tlskey=/home/user/.docker/server-  key.pem --userns-remap="default" -H tcp://0.0.0.0:2376

    # Restart daemon and service
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    sudo systemctl status docker

    cat /etc/subuid
    ```
  - Now every container will run as the low privileged dockremap user.
  - However, the user **within the container will be root**.
    ```
    docker run -it -rm <image_id> /bin/bash
    $~ whoami
    root
    ```

# Securing the Docker Container

## 1. Disable root user within the container
- For this, you need to create your own Docker image
- A sample Dockerfile that creates a low privileged user `lowuser` and disables login as `root`
  ```
  FROM ubuntu

  RUN groupadd -r lowuser && useradd -r -g lowuser lowuser
  RUN chsh -s /usr/sbin/nologin root 

  # Environment variables
  ENV HOME /home/lowuser
  ENV DEBIAN_FRONTEND=noninteractive
  ```
- Build the Dockerfile
  ```
  docker build . -t lowusertest
  ```
- Start the container
  ```
  docker run -u lowuser -it --rm <image_id> /bin/bash 
  ```
- Switching to a root user within this container will be prohibited

## 2. Prevent privilege escalation
- Sometimes SUID binaries can be abused to gain root privileges
- To perevent this start the container using the following command
  ```
  docker run -u lowuser -it --rm --security-opt=no-new-privileges <image_id> /bin/bash 
  ```

## 3. Assign selected Linux kernel capabilities
- **Capabilities**: In simple terms, instead of assigning full root privileges, capabilities allow assigning selected privileges of the root user.
- Example, we can run the container as lowuser, but with `CAP_NET_ADMIN` capability, which allows lowuser to perform network administration operations. Make sure to use `--cap-drop all` to disable other capabilities by default
  ```
  docker run --cap-drop all --cap-ad NET_ADMIN -u lowuser -it --rm <image_id> /bin/bash 
  ```

## 4. Make the filesystem read only
- If you want to make the whole file system read only (applies to all users, even root)
  ```
  docker run --read-only -it -rm <image_id> /bin/bash
  ```
- If you want to make a temporary writeable directory but keep the rest of the file system read only (applies to all users, even root)
  ```
  docker run --read-only --tmpfs /opt -it -rm <image_id> /bin/bash
  ```

## 5. Disable inter container communication (icc)
- See docker networks - By default all containers use the `bridge` network. So we have to create our own network with icc disabled
  ```
  docker network ls
  ```
- See inter container communication setting of a network
  ```
  docker inspect <network_name>
  ```
- Create our own network with icc disabled
  ```
  docker network create --driver bridge -o "com.docker.network.bridge.enable_icc"="false" test-net
  docker inspect test-net
  ```
- Now start containers with this new network
  ```
  docker run --network test-net -it --rm <image_id> /bin/bash
  ```

# Auditing Docker

# Docker Attacks

**Resources:**
1. [Securing The Docker Daemon by HackerSploit](https://www.youtube.com/watch?v=70QOBVwLyC0&list=PLBf0hzazHTGNv0-GVWZoveC49pIDHEHbn&index=7)
2. [How To Secure & Harden Docker Containers by HackerSploit](https://www.youtube.com/watch?v=CQLtT_qeB40&list=PLBf0hzazHTGNv0-GVWZoveC49pIDHEHbn&index=6)

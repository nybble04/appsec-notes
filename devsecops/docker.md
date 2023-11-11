# Securing the Docker Daemon
Regular port: 2375

TLS port: 2376

## 1. Secure communication
- **We need to encrypt the communication between the docker client and the server**

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

## 2. Implementing Namespaces
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

# Securing the Docker Container

# Auditing Docker

# Docker Attacks

**Resources:**
1. [Securing The Docker Daemon by HackerSploit](https://www.youtube.com/watch?v=70QOBVwLyC0&list=PLBf0hzazHTGNv0-GVWZoveC49pIDHEHbn&index=7)

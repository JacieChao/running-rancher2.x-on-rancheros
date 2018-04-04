# os-rancher2.0-compatibility[WIP]
As Rancher2.0 is about to be released soon, this document will guide users to start Rancher2.0 on RancherOS, while also recording RancherOS's incompatibility with Rancher2.0.

# Goals
- Start Rancher2.0 on RancherOS
  - [Requirements](#requirements)
  - [Getting Started](#getting-started)
- Recording incompatibility between RancherOS and Rancher2.0

# Start Rancher2.0 on RancherOS
This Section will show you how to deploy Rancher2.0 on RancherOS. In our example, we use the aws virtual machine with the x86_64 architecture.

## Requirements ##
- [RancherOS-v1.3.0](https://github.com/rancher/os/releases/v1.3.0)
- [Rancher-v2.0.0](https://github.com/rancher/rancher/master) which using docker image [rancher/server:master](https://hub.docker.com/r/rancher/server/tags/) instead, temporary.
- [Rke-v0.1.4](https://github.com/rancher/rke/releases/v0.1.4)

## Getting Started ##
In our example, we use 3 ros nodes(t2.medium on aws). 
One of the nodes is used as both the etcd role and the controlplane role, others are used as both the etcd role and the worker role. 
Notice that we deploy our rancher-server on controlplane node.

**Deploy Rancher2.0**

To install Rancher on your host, connect to it and then use a shell to install.
```bash
sudo docker run -d --restart=unless-stopped -p 8080:80 -p 8443:443 rancher/server:master
```
Open a web browser and enter the IP address of your host and replace <SERVER_IP> with your host IP address:
```bash
https://<SERVER_IP>:8080
```






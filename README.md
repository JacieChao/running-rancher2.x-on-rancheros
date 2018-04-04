# os-rancher2.0-compatibility[WIP]
As Rancher2.0 is about to be released soon, this document will guide users to start Rancher2.0 on RancherOS, while also recording RancherOS's incompatibility with Rancher2.0.

# Goals
- [Start Rancher on RancherOS](#start-rancher-on-rancheros)
  - [Requirements](#requirements)
  - [Getting Started](#getting-started)
    - [Deploy Rancher](#deploy-rancher)
    - [Using Rke deploy cluster](#using-rke-deploy-cluster)
- [Recording incompatibility between RancherOS and Rancher](#recording-incompatibility-between-rancheros-and-rancher)

# Start Rancher on RancherOS #
This Section will show you how to deploy Rancher2.0 on RancherOS. In our example, we use the aws virtual machine with the x86_64 architecture.

## Requirements ##
- [RancherOS-v1.3.0](https://github.com/rancher/os/releases/v1.3.0)
- [Rancher-v2.0.0](https://github.com/rancher/rancher/master) which using docker image [rancher/server:master](https://hub.docker.com/r/rancher/server/tags/) instead, temporary.
- [Rke-v0.1.4](https://github.com/rancher/rke/releases/v0.1.4)

## Getting Started ##
In our example, we use 3 ros nodes(t2.medium on aws). 
One of the nodes is used as both the etcd role and the controlplane role, others are used as both the etcd role and the worker role. 
Notice that we deploy our rancher-server on controlplane node.

### Deploy Rancher ###

To install Rancher on your host, connect to it and then use a shell to install.
```bash
sudo docker run -d --restart=unless-stopped -p 8080:80 -p 8443:443 rancher/server:master
```
Open a web browser and enter the IP address of your host and replace <SERVER_IP> with your host IP address:
```bash
https://<SERVER_IP>:8443
```

### Using Rke deploy cluster ###

Download rke binary:
```bash
wget -O rke https://github.com/rancher/rke/releases/download/v0.1.4/rke_linux-amd64 && chmod +x rke
```

Prepare cluster.yml:
```yaml
---
nodes:
  - address: <address>
    user: rancher
    role:
    - controlplane
    - etcd
    ssh_key_path: /home/rancher/.ssh/id_rsa
  - address: <address>
    user: rancher 
    role:
    - worker
    - etcd
    ssh_key_path: /home/rancher/.ssh/id_rsa
  - address: <address>
    user: rancher 
    role:
    - worker
    - etcd
    ssh_key_path: /home/rancher/.ssh/id_rsa
services:
  kube-api:
    service_cluster_ip_range: 10.233.0.0/18
    pod_security_policy: false
    extra_args:
      v: 4
  kube-controller:
    cluster_cidr: 10.233.64.0/18
    service_cluster_ip_range: 10.233.0.0/18
  scheduler:
  kubelet:
    cluster_domain: cluster.local
    cluster_dns_server: 10.233.0.3
    infra_container_image: gcr.io/google_containers/pause-amd64:3.0
  kubeproxy:

# supported plugins are:
# flannel
# calico
# canal
# weave
#
# If you are using calico on AWS or GCE, use the network plugin config option:
# 'calico_cloud_provider: aws'
# or
# 'calico_cloud_provider: gce'
network:
  plugin: flannel 
  options:
    flannel_image: quay.io/coreos/flannel:v0.9.1
    flannel_cni_image: quay.io/coreos/flannel-cni:v0.2.0

authentication:
  strategy: x509

# all addon manifests MUST specify a namespace
addons: |-
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
      namespace: default
    spec:
      containers:
      - name: busybox
        image: busybox:musl
        command:
          - sleep
          - "3600"
        imagePullPolicy: IfNotPresent
      restartPolicy: Always
system_images:
  etcd: rancher/etcd:v3.0.17
  kubernetes: rancher/k8s:v1.8.7-rancher1-1
  alpine: alpine:latest
  nginx_proxy: rancher/rke-nginx-proxy:v0.1.1
  cert_downloader: rancher/rke-cert-deployer:v0.1.1
  kubernetes_services_sidecar: rancher/rke-service-sidekick:v0.1.0
  kubedns: rancher/k8s-dns-kube-dns-amd64:1.14.5
  dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.14.5
  kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.14.5
  kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0

ssh_key_path: ~/.ssh/test

# Kubernetes authorization mode
# Use `mode: rbac` to enable RBAC
# Use `mode: none` to disable authorization
authorization:
  mode: rbac

# If set to true, rke won't fail when unsupported Docker version is found
ignore_docker_version: false

kubernetes_version: v1.8.7-rancher1-1

# List of registry credentials, if you are using a Docker Hub registry,
# you can omit the `url` or set it to `docker.io`
private_registries:
  - url: <repo_url>
    user: <username>
    password: <password>
```

Change all nodes' ros Docker engine to the version which rke recommendation, you can use `ros engine list` command to show which Docker engines are available to switch to:
```bash
ros engine list
ros engine switch docker-17.03.2-ce
```

Standing up a Kubernetes is as simple as creating a cluster.yml configuration file and running the command:
```bash
./rke up --config cluster.yml
```

# Recording incompatibility between RancherOS and Rancher #
This Section will record incompatibility between RancherOS and Rancher2.0.







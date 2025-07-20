---
title: "Deploying a Kubernetes Cluster Using kubeadm"
description: "Using kubeadm and CRI-O to deploy a Kubernetes cluster"
date: 2023-09-05T00:59:00+10:00
lastmod: 2023-11-16T18:22:00+10:00
categories:
  - Learning
tags:
  - Linux
  - Kubernetes
---

## Installing CRI-O

This guide uses CRI-O version **1.28**, which can be set with `export VERSION=1.28`.

### On Ubuntu 22.04 (Jammy)

You’ll need to add the appropriate APT repositories, import GPG keys, and install `cri-o`, `cri-o-runc`, and `containernetworking-plugins`. Ensure CRI-O is enabled and running using `systemctl`.

```shell
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4D64390375060AA4
export OS=xUbuntu_22.04
export VERSION=1.28
rm /usr/share/keyrings/libcontainers-archive-keyring.gpg
rm /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/libcontainers-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb [signed-by=/usr/share/keyrings/libcontainers-crio-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

mkdir -p /usr/share/keyrings
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-archive-keyring.gpg
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/Release.key | gpg --dearmor -o /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg

apt-get update
apt-get install -y cri-o cri-o-runc containernetworking-plugins

systemctl enable crio
systemctl start crio
systemctl status crio
```

### On CentOS 7

Add the relevant YUM repositories for your CRI-O version and OS, then install the same packages. Again, make sure to enable and start the `crio` service.

```shell
export OS=CentOS_7
export VERSION=1.28

curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
yum install -y cri-o containernetworking-plugins

systemctl enable crio
systemctl start crio
systemctl status crio
```

## Configuring CRI-O

Update CRI-O’s root directory (e.g., to `/home/crio`) and configure the `pause_image` (e.g., from Alibaba Cloud's registry). Restart the CRI-O service to apply changes.

```shell
mkdir /var/lib/crio
mkdir /home/crio
cat << EOF | sudo tee /etc/crio/crio.conf
[crio]
root = "/home/crio"
[crio.api]

[crio.runtime]

[crio.image]
pause_image = "registry.aliyuncs.com/google_containers/pause:3.9"
[crio.network]

[crio.metrics]

[crio.tracing]

[crio.nri]

[crio.stats]

EOF
systemctl restart crio
```

## Setting Up CNI (Container Network Interface)

Create a CNI bridge network configuration file. You’ll need to adjust the `subnet` value (e.g., `10.9.10.0/24`) to fit your network design.

```shell
mkdir /etc/cni/net.d
cat << EOF | sudo tee /etc/cni/net.d/100-crio-bridge.conflist
{
  "cniVersion": "1.0.0",
  "name": "crio",
  "plugins": [
    {
      "type": "bridge",
      "mtu": 1420,
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "hairpinMode": true,
      "ipam": {
        "type": "host-local",
        "routes": [
            { "dst": "0.0.0.0/0" }
        ],
        "ranges": [
            [{ "subnet": "10.9.10.0/24" }]
        ]
      }
    }
  ]
}
EOF
```

## Installing kubeadm, kubelet, and kubectl

### On Ubuntu

Install version **1.26** of Kubernetes tools by adding the Kubernetes APT repo and holding package versions to prevent unintended upgrades.

```shell
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.26/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### On CentOS

Disable SELinux temporarily and configure the Kubernetes repo. Install the components using YUM. The version can be pinned if needed (e.g., `yum install kubelet-1.26.8-0  kubeadm-1.26.8-0 kubectl-1.26.8-0 -y`).

```shell
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

## Configuring kubelet

Adjust the kubelet configuration to include extra arguments such as `--node-ip` and `--root-dir`. This ensures proper node identification and storage path configuration.

```shell
cat << EOF | sudo tee /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
EOF
cat << EOF | sudo tee /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--root-dir=/home/kubelet --fail-swap-on=false --node-ip=10.9.0.12"
EOF
```

## Initialising the Control Plane (on the Master Node)

Run `kubeadm init` with flags for:

- CRI-O socket path
- Image repository (e.g., Alibaba Cloud mirror)
- Advertised IP of the API server
- Pod network CIDR

```shell
kubeadm init --cri-socket unix:///var/run/crio/crio.sock --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=10.9.0.10 --pod-network-cidr=10.9.0.0/16
```

## Resetting the Cluster

Use `kubeadm reset` with the CRI-O socket if you need to tear down and re-initialise the cluster.

```shell
kubeadm reset --cri-socket unix:///var/run/crio/crio.sock
```

## Joining Other Nodes to the Cluster

Worker nodes can join the cluster using the `kubeadm join` command provided by the `kubeadm init` output. Remember to use the correct `--token` and `--discovery-token-ca-cert-hash`.

```shell
kubeadm join 10.9.0.10:6443 --cri-socket unix:///var/run/crio/crio.sock --token 5riq16.lqmfkw94eff6ovo2 --discovery-token-ca-cert-hash sha256:9589d683977ce5511f4cf61912b17bef41ab9696ab2d5664fa94310910811387
```

## Enabling Scheduling on the Master Node

To allow workloads to run on the control plane node, remove the default taint:

```shell
kubectl taint no [主节点名字] node-role.kubernetes.io/control-plane:NoSchedule-
```

## Adding More Nodes to the Cluster

To scale your cluster, deploy additional nodes using the same steps. Then, on the master node, create a new join token:

```shell
kubeadm token create
```

Use the updated `kubeadm join` command to integrate new nodes.

## Caveats for CentOS 7

* The CNI plugin binaries might be missing under `/opt/cni/bin/`. You may need to install them manually.

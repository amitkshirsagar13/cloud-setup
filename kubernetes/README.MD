
sudo systemctl disable firewalld iptables iptables-service iptables-services && sudo systemctl stop firewalld iptables iptables-service iptables-services



sudo vi /etc/ssh/ssh_config
sudo systemctl enable --now sshd && 

Enable Bridged Adaptor Networking:

Kubernetes Master Node: Setup
kube-master:



Determine what IP want to use for Master:
192.168.1.150 kube-master

[master@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE=Ethernet
BOOTPROTO=static
NAME=enp0s8
UUID=3e0f5ae2-7e70-373a-9a41-1512a4b13b7c
ONBOOT=yes
AUTOCONNECT_PRIORITY=-999
DEVICE=enp0s8
DNS1=8.8.8.8
DNS2=4.4.4.4
GATEWAY=192.168.50.1
HOSTNAME=master.shinigami.io
IPADDR=192.168.50.100
NETMASK=255.255.255.0
NM_CONTROLLED=yes
ONBOOT=yes
[master@localhost ~]$






sudo dnf install cockpit cockpit-packagekit cockpit-docker cockpit-machines cockpit-ostree cockpit-selinux cockpit-kubernetes flannel kubernetes docker ntp etcd 

Modify Below Configuration:

sudo vi /etc/etcd/etcd.conf
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"

sudo systemctl enable --now etcd 
etcdctl mk /shinigami.io/network/config '{"Network":"172.17.0.0/16"}'

sudo vi /etc/kubernetes/apiserver
KUBE_API_ADDRESS="--address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBELET_PORT="--kubelet_port=10250"
KUBE_ETCD_SERVERS="--etcd_servers=http://kube-master:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

sudo vi /etc/sysconfig/flanneld
FLANNEL_ETCD="http://kube-master:2379"
FLANNEL_ETCD_KEY="/shinigami.io/network"

sudo vi /etc/kubernetes/config
KUBE_MASTER="--master=http://kube-master:8080"

sudo vi /etc/kubernetes/kubelet
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
KUBELET_HOSTNAME="--hostname_override=kube-minion1"
KUBELET_API_SERVER="--api_servers=http://kube-master:8080"
KUBELET_ARGS=""

sudo systemctl enable --now kube-proxy kubelet docker flanneld ntpd kube-apiserver kube-controller-manager kube-scheduler etcd cockpit.socket
sudo systemctl restart --now kube-proxy kubelet docker flanneld ntpd kube-apiserver kube-controller-manager kube-scheduler etcd cockpit.socket

Kubernetes Minion Node: Setup

We want to distinctly range IP Addresses to allow handling nodes from multiple machines.
192.168.1.151 kube-minion1 
192.168.1.161 kube-minion2 


sudo dnf install cockpit cockpit-ostree cockpit-packagekit cockpit-docker cockpit-machines cockpit-selinux cockpit-kubernetes flannel kubernetes docker ntp


sudo vi /etc/sysconfig/flanneld
FLANNEL_ETCD="http://kube-master:2379"
FLANNEL_ETCD_KEY="/shinigami.io/network"

sudo vi /etc/kubernetes/config
KUBE_MASTER="--master=http://kube-master:8080"

sudo vi /etc/kubernetes/kubelet
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
KUBELET_HOSTNAME="--hostname_override=kube-minion1"
KUBELET_API_SERVER="--api_servers=http://kube-master:8080"
KUBELET_ARGS=""



sudo systemctl enable --now kube-proxy kubelet docker flanneld ntpd cockpit.socket
sudo systemctl restart --now kube-proxy kubelet docker flanneld ntpd cockpit.socket



Solve Volume Mount issue:
ls -laZ /home/master/shared
chcon -Rt svirt_sandbox_file_t /home/master/shared

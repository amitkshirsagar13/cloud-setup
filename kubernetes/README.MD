
sudo systemctl disable firewalld iptables iptables-service iptables-services && sudo systemctl stop firewalld iptables iptables-service iptables-services

sudo vi /etc/ssh/ssh_config
sudo systemctl enable --now sshd && sudo systemctl restart etcd  

Enable Bridged Adaptor Networking:

Kubernetes Master Node: Setup
kube-master:

Determine what IP want to use for Master:
192.168.1.150 kube-master
sudo dnf install cockpit cockpit-ostree cockpit-packagekit cockpit-docker cockpit-machines cockpit-selinux flannel kubernetes docker ntp etcd 

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

sudo systemctl enable --now kube-proxy kubelet docker flanneld ntpd kube-apiserver kube-controller-manager kube-scheduler etcd
sudo systemctl restart --now kube-proxy kubelet docker flanneld ntpd kube-apiserver kube-controller-manager kube-scheduler etcd

Kubernetes Minion Node: Setup

We want to distinctly range IP Addresses to allow handling nodes from multiple machines.
192.168.1.151 kube-minion1 
192.168.1.161 kube-minion2 


sudo dnf install cockpit cockpit-ostree cockpit-packagekit cockpit-docker cockpit-machines cockpit-selinux flannel kubernetes docker ntp


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



sudo systemctl enable --now kube-proxy kubelet docker flanneld ntpd 
sudo systemctl restart --now kube-proxy kubelet docker flanneld ntpd 



Solve Volume Mount issue:
ls -laZ /home/master/shared
chcon -Rt svirt_sandbox_file_t /home/master/shared
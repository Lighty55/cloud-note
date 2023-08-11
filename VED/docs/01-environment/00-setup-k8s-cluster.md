#####Using Kubeadm
- If using offline environment, please download packages requires for installation
- Pre-Installation
```
sudo swapoff -a 
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
```
sudo modprobe overlay
sudo modprobe br_netfilter
sudo sysctl --system
```
`adding nameserver in /etc/resolv.conf ---- nameserver <ip-address>`


#####Online enviroment
- Install docker
cd /root/download/docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
systemctl start docker
systemctl enable docker
rm /etc/containerd/config.toml
systemctl restart containerd
```
- Install kube-package
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
- Init controlplane `sudo kubeadm init --control-plane-endpoint <master-node-ip> --upload-certs -v=9`
- Join node with command generated from `kubeadm init`
- Adding network - using calico (on all nodes / apply `calico.yaml` on init master node only)
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
kubeclt apply -f calico.yaml
```


#####Offline enviroment
- Install docker
cd /root/download/docker
```
rpm -ivh --replacefiles --replacepkgs *.rpm --force
systemctl enable docker
systemctl start docker
rm /etc/containerd/config.toml
systemctl restart containerd
```
- Install kube-package
```
cd /root/download/kube/k8s
rpm -ivh --replacefiles --replacepkgs *.rpm --force
systemctl enable kubelet
```
- Upload local kubeadm images (on all nodes)
```
cd /root/download/kubeadm-imgs
ls * | while read image; do sudo ctr -n k8s.io images import $image; done 
```
- Init controlplane `sudo kubeadm init --control-plane-endpoint <master-node-ip> --upload-certs -v=9`
- Join node with command generated from `kubeadm init`
- Adding network - using calico (on all nodes - apply `calico.yaml` on init master node only)
```
cd /root/download/calico
ls * | while read image; do sudo ctr -n k8s.io images import $image; done
kubeclt apply -f calico.yaml
```
set `ImagePullPolicy: Never` for `calico.yaml`


###Join Controlplane/Worker 
#example
```
#Master Node
kubeadm join 172.16.68.192 --token e4gwb0.red7b8wb29fk6zfg \
        --discovery-token-ca-cert-hash sha256:66d110f8963149bb6d61bc1aa1a6c809ed2cc0915271e7684324c57441b71434 \
        --control-plane --certificate-key 1f8e3327248ced55451905c83e59cfe41fdc109e262254ae41f1dc0297e632fc
#Worker Node
kubeadm join 172.16.68.192 --token e4gwb0.red7b8wb29fk6zfg \
        --discovery-token-ca-cert-hash sha256:66d110f8963149bb6d61bc1aa1a6c809ed2cc0915271e7684324c57441b71434
```

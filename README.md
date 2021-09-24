# Install-Kubernetes
Tutorial Install Kubernetes

![Product Name Screen Shot](https://logos-download.com/wp-content/uploads/2018/09/Kubernetes_Logo.png)

# Housekeeping

```
yum update -y
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
systemctl disable firewalld
systemctl stop firewalld
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
Housekeeping done selesai

# Tambahakan ip addres dan hostname di kesemua server kubernetes
```
vim /etc/hosts
```

- 192.168.1.10 kubeadm.master.qs
- 192.168.1.11 kubeworker.node1.qs
- 192.168.1.12 kubeworker.node2.qs

```
systemctl restart NetworkManager
```

- ping 192.168.1.10
- ping 192.168.1.10
- ping 192.168.1.10

- ping 8.8.8.8

```
systemctl stop firewalld
systemctl disable firewalld
```

# Install Docker

```
yum install -y yum-utils device-mapper-persistent-data lvm2 yum-plugin-versionlock
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
```

```
mkdir /etc/docker
```

```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

```
mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl enable docker
groupadd docker
MAINUSER=$(logname)
usermod -aG docker $MAINUSER
systemctl start docker
yum versionlock docker-ce docker-ce-cli containerd.io
```
Selesai Install Docker
  
# Install K8s Stuff

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
sudo sysctl --system
```

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

# Mengatur SELinux menjadi permissive mode (menonaktifkannya secara efektif)

```
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
  
# Restart the kubelet daemon to reload the configuration

```
systemctl daemon-reload
systemctl restart kubelet
```

# Verifikasi kita sudah punya semua images kubernetes
```
kubeadm config images pull
```

# -----------------------------------------------------------------------------------------------------------------------------------------------------------  
# Mulai dari Sini Hanya Untuk Installasi Kubernetes Master

# Initialisasi kubeadm
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/24
```

# kubectl dapat diakses oleh user biasa
```
mkdir -p /home/$MAINUSER/.kube
cp -i /etc/kubernetes/admin.conf /home/$MAINUSER/.kube/config
chown -R ${MAINUSER}:${MAINUSER} /home/${MAINUSER}/.kube
```

# simpan command untuk worker anda join kedalam master kubernetes
kubeadm join 192.168.0.120:6443 --token khm95w.mo0wwenu2o9hglls --discovery-token-ca-cert-hash sha256:aeb0ca593b63c8d674719858fd2397825825cebc552e3c165f00edb9671d6e32
  
# Menginstal add-on jaringan Pod DNS klaster (CoreDNS) tidak akan menyala sebelum jaringan dipasangkan.
```
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

# Menggabungkan Node-Node Pada kubernetes Cluster
Node adalah tempat beban kerja (Container, Pod, dan lain-lain) berjalan. Untuk menambahkan Node baru pada klaster lakukan hal berikut pada setiap mesin:
  
# kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
 
- Install kubeadm selesai

# Kumpulan Command Kubernetes

- melihat node yang aktif
  
```
kubectl get nodes
```

- melihat namespaces
  
```
kubectl get namespace -o wide
```

- melihat pods
  
```
kubectl get pods --all-namespaces
```
  
- melihat service
  
```
kubectl get service --all-namespaces
```
  
- melihat service
  
```
kubectl get service
```

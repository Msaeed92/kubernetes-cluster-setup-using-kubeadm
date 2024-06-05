Follow the below steps for creating Kubernetes Cluster on Rocky Os.

**Create 3 Virtual machine ( Rocky OS server mode) with the below resources**

HDD : 30 GB.

Memory : 3 GB.

CPU : 1 cpu for worker nodes --- 2 cpu for master node (control-plane).

Steps 1 to 9 is done on all nodes. Steps 10 and 11 only on Master . Step 12 only on Node1 & Node2

-------------------------------------------------------

**Step1 - Set Hostnames**

hostnamectl set-hostname Master (On Master)<br />
hostnamectl set-hostname Node1(On Node1)<br />
hostnamectl set-hostname Node2 (On Node2)<br />

------------------------------------------------------

**Step2 - Configure Network and Ip's**

Run nmtui con to indentify the network details.<br />

------------------------------------------------------------

**Step3 - Edit /etc/hosts file**

Run the below commands on the machines. Change the IP address and host name as per your machine settings.<br />
cat << EOF >> /etc/hosts<br />

192.168.0.xxx Master<br />
192.168.0.xxx Node1<br />
192.168.0.xxx Node2<br />

EOF
  
-----------------------------------------------------------
  
**Step4 - Disable SELinux**
  
setenforce 0<br />
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux<br />

-----------------------------------------------------------
  
**Step5 - Disable firewall and edit Iptables settings**
  
systemctl disable firewalld<br />
modprobe br_netfilter<br />
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables<br />
  
----------------------------------------------------------
  
**Step6 - Setup Kubernetes Repo**

cat << EOF > //etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/

enabled=1

gpgcheck=1

gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key

EOF<br />

---------------------------------------------------------
  
**Step7 - Installing Kubeadm, Enable and start the services**
  
yum install kubeadm -y<br />
systemctl enable kubelet --now <br />
  
--------------------------------------------------------

**Step 8- Installing Docker, Enable and start the services**
Setup Docker Repo

dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo <br /> 

Installing

yum install docker-ce docker-ce-cli containerd.io -y <br />
systemctl enable docker --now <br />

-----------------------------------------------------

**Step9 - Disable Swap**
  
swapoff -a<br />
vi /etc/fstab and Comment the line with Swap Keyword<br />
  
-----------------------------------------------------
  
**Step10 - Initialize Kubernetes Cluster**

kubeadm init<br />
mkdir -p $HOME/.kube<br />
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config<br />
chown $ (id -u): $ (id -g) $HOME/.kube/config
  
------------------------------------------------------
  
**Step11 - Installing Pod Network using Calico network**

kubectl create -f https://docs.projectcalico.org/manifests/calico.yaml<br />
kubectl get pods -n kube-system<br />

----------------------------------------------------
**Step12 - Join Worker Nodes**
  
  Use the token from kubeadm token create --print-join-command <br />
  kubeadm token create --print-join-command<br />
  run the output on Node1 and Node2<br />
  
  output should be as the below :
  
  kubeadm join 192.168.0.xxx:6443 --token XXX\--discovery-token-ca-cert-hash sha256:XX<br />

  ---------------------------------------------------
  Troubleshooting - if you faced error ( --ignore-preflight-errors) while kubeadmin init
  
  1- rm -f /etc/containerd/config.toml
  
  2- systemctl restart containerd
  
  3- kubeadm init
  

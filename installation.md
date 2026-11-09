### **install RKE with ansible**

config pre install
```
sudo hostnamectl set-hostname rke2-server
sudo hostnamectl set-hostname rke2-worker

edit host ke2 VM
sudo nano /etc/hosts
192.168.10.11 rke2-server
192.168.10.12 rke2-worker
```
### **ssh-key public access**
hanya di rke2-server saja
```
ssh-keygen -t ed25519
ssh-copy-id ubuntu@192.168.10.11
ssh-copy-id ubuntu@192.168.10.12
```
melakuka install provisioning ansible dan kebutuhan lain
```
sudo apt update -y
sudo apt install -y \
  ansible \
  git \
  python3 \
  python3-pip \
  python3-venv \
  openssh-client \
  curl \
  wget
```
cek provisioning
```
ansible --version
git --version
python3 --version
```
buat folder untuk rke-deployment
```
mkdir -p ~/rke2-deployment
cd ~/rke2-deployment
git clone https://github.com/ranchergovernment/rke2-ansible.git
cd rke2-ansible
```
melakukan config list node untuk di remote
```
mkdir -p inventory
nano inventory/hosts.yml
```
masukan config berikut
```
---
rke2_cluster:
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: /home/ubuntu/.ssh/id_ed25519

  children:
    rke2_servers:
      hosts:
        rke2-server:
          ansible_host: 192.168.10.11

    rke2_agents:
      hosts:
        rke2-worker:
          ansible_host: 192.168.10.12

```

validasi test1
```
ansible-inventory \
  -i inventory/hosts.yml \
  --graph
```
validasi test2
```
ansible rke2_cluster \
  -i inventory/hosts.yml \
  -m ping
```
### **installasi RKE**
```
ansible-playbook \
  ./site.yml \
  -i inventory/hosts.yml \
  -b
```
cek service di rke-server
```
sudo systemctl status rke2-server
```
cek service  di rke-worker
```
sudo systemctl status rke2-agent
```
verifikasi kubectl
```
echo 'export PATH=$PATH:/var/lib/rancher/rke2/bin' \
  >> ~/.bashrc

source ~/.bashrc
```
cek config kubectl
```
sudo kubectl \
  --kubeconfig /etc/rancher/rke2/rke2.yaml \
  get nodes -o wide
```
```
kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get nodes -o wide
kubectl get nodes -o wide
```
 


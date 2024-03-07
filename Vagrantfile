$script = <<-SCRIPT
echo I am provisioning...
date > /etc/vagrant_provisioned_at

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"


#sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"


sudo apt update -y

### Instalar Docker ###

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker


### Instalar Kubernetes Kubeadm ###
cd /home/vagrant

### Deshabilitar Swap ###
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

### Parametros del Kernel ###
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

### Containerd Runtime ###
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo apt install -y containerd.io

### Configurar Containerd ###
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

### Repo Kubernetes 1.29 ###
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

### Instalar Kubelet Kubeadm y Kubectl ###
sudo apt-get update -y
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
#sudo kubeadm init --node-name node1
#mkdir -p $HOME/.kube
#sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#sudo chown $(id -u):$(id -g) $HOME/.kube/config

### Instalar plugin de Red ###
#kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
#kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
#kubectl get pods -n kube-system

### Cambiar Hostname ###
hostnamectl hostname node1

### Unir al Cluster ###
kubeadm join 10.211.55.33:6443 --token 2scg99.agg3lpx4oic10j55 --discovery-token-ca-cert-hash sha256:30fd88b6854e588c01a4db6b947b6c591775bd6ffc65a10d81a644f5ca7cf4d5


ip a
#echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC4qWRwFjlXm7keNSV+SKu4oihLGSZH4q9DFHV6dHJeg94fR8DLnCZw1MFARgSQFXGlZwXMjok8O43k+qSXnzg+n8dHUU2Pb3Mt7rVpvRzMiK/xQyPvFP5vOjS9zUiOG98d4LvN9CuEeCcWdbTk+rMlNF+uqivMg1MgE5YZ5W4A2trErOSLQocL4qgKEb9N+7UXO5ln1VRRcA/ceiXF3rh4NC5lgsBGXcB57QgF1G2WbKgfrRKo6EP/ph7s5zqig3OcTPwU00bhuhvy0Xcn3jkORvKNR5th/ynA9O98/DxgzMctQyYvOHms7fGN3pUI+24sXxINLoznjMpHKDl7R7P4XdL5vgXduDH+JtrHsDV68VrOVPE3oyDNaFjptqF4Ppav5iNHYiD1sB7SjtN4ygO0FuAmDqGeHNLwoyzuhcwwYmhjUl+gSh9I6a7jdYN9NvA5oRMU3fRdMrEgOYX6b5MdXBdC/jW93gWzVThQdaKLxio2j2Nq7ttxyjVfXbEyCwk= Laske@DESKTOP-WINABNQ" > .ssh/authorized_keys
SCRIPT
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"  # Utilizamos una imagen ligera de Ubuntu 20.04

  # Configuración específica para VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"  # 2 GB de memoria RAM
    vb.cpus = 2         # 3 cores de procesador
  end

  # Configuración del disco duro (20 GB)
  config.vm.disk :disk, size: "20GB", name: "gyptazy/ubuntu22.04-arm64-node1"
  #config.vm.network "public_network", dhcp: true # Utiliza una red pública
  config.vm.network "private_network", ip: "192.168.144.166"
  
  # Provisión adicional (opcional) para configurar tu máquina virtual
   config.vm.provision "shell", inline: $script   

end

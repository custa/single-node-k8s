# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/xenial64"
  config.vm.host_name = "ubuntu-k8s"

  # 设置代理
  #config.proxy.http     = "http://10.0.2.2:8080"
  #config.proxy.https    = "http://10.0.2.2:8080"

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
set -xe
export PS4='+[$LINENO]'


# no_proxy
echo 'export no_proxy=$(echo 10.0.2.{1..255} | sed "s/ /,/g")' >/etc/profile.d/no_proxy.sh
echo 'export no_proxy=${no_proxy},localhost,127.0.0.1' >>/etc/profile.d/no_proxy.sh
source /etc/profile.d/no_proxy.sh &>/dev/null


cp -an /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt.cyp
cat >>/etc/ssl/certs/ca-certificates.crt <<\EOF

EOF


# 参考 https://kubernetes.io/docs/setup/independent/install-kubeadm/
# 选择1： Docker from Ubuntu’s repositories
#apt-get update
#apt-get install -y docker.io

# 参考 https://kubernetes.io/docs/setup/independent/install-kubeadm/ 及 https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/
# 选择2： Docker CE 17.09 from Docker’s repositories
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSLk https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.09 | head -1 | awk '{print $3}')

# 配置 docker HTTP 代理
mkdir -p /etc/systemd/system/docker.service.d
echo '[Service]' >/etc/systemd/system/docker.service.d/http-proxy.conf
echo "Environment='HTTP_PROXY=${http_proxy}'" >>/etc/systemd/system/docker.service.d/http-proxy.conf
systemctl daemon-reload
systemctl restart docker

# kubeadm, kubelet and kubectl
apt-get install -y apt-transport-https
curl -sk https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl


# 参考 https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
kubeadm init

mkdir -p /root/.kube
cp /etc/kubernetes/admin.conf /root/.kube/config

usermod -aG docker ubuntu
mkdir -p /home/ubuntu/.kube
cp /etc/kubernetes/admin.conf /home/ubuntu/.kube/config
chown -R ubuntu: /home/ubuntu/.kube

  SHELL
end

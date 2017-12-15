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
  config.vm.box = "centos/7"
  config.vm.host_name = "centos-k8s"

  # 设置代理
  #config.proxy.http     = "http://10.0.2.2:8080"
  #config.proxy.https    = "http://10.0.2.2:8080"

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder ".", "/share"

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


# 解决 -bash: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory
cat >/etc/environment <<\EOF
LANG=en_US.utf-8
LC_ALL=en_US.utf-8
EOF


# no_proxy
echo 'export no_proxy=$(echo 10.0.2.{1..255} | sed "s/ /,/g")' >/etc/profile.d/no_proxy.sh
echo 'export no_proxy=${no_proxy},localhost,127.0.0.1' >>/etc/profile.d/no_proxy.sh
source /etc/profile.d/no_proxy.sh &>/dev/null


cp -an /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem.cyp
cat >>/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem <<\EOF

EOF


# 安装 etcd 和 Kubernetes
yum -y install etcd kubernetes
usermod -aG root vagrant


# 配置 docker HTTP 代理
#cp -an /etc/sysconfig/docker /etc/sysconfig/docker.cyp
#sed -i "/^HTTP_PROXY=/d" /etc/sysconfig/docker
#echo "HTTP_PROXY='${http_proxy}'" >>/etc/sysconfig/docker


# 创建并配置 Service Account 相关密钥，解决 kubectl create 创建 pod 出错问题
openssl genrsa -out /etc/kubernetes/serviceaccount.key 4096

cp -an /etc/kubernetes/apiserver /etc/kubernetes/apiserver.cyp
# --service-account-key-file 不配置不影响
#sed -ri 's|^KUBE_API_ARGS="([^"]*)"|KUBE_API_ARGS="\\1 --service-account-key-file=/etc/kubernetes/serviceaccount.key"|' /etc/kubernetes/apiserver

cp -an /etc/kubernetes/controller-manager /etc/kubernetes/controller-manager.cyp
sed -ri 's|^KUBE_CONTROLLER_MANAGER_ARGS="([^"]*)"|KUBE_CONTROLLER_MANAGER_ARGS="\\1 --service-account-private-key-file=/etc/kubernetes/serviceaccount.key"|' /etc/kubernetes/controller-manager
# 注：配置 --service-account-private-key-file 后，kube-controller-manager 会向 etcd 的 /registry/serviceaccounts/default/default 和 /registry/serviceaccounts/kube-system/default 写入 "selfLink" 和 "secrets"，即使后续去掉 --service-account-private-key-file，etcd 的数据不会自动删除


# 规避拉取 registry.access.redhat.com/rhel7/pod-infrastructure:latest 失败问题
rm -f /etc/docker/certs.d/*/redhat-ca.crt


systemctl enable etcd kube-apiserver kube-controller-manager kube-proxy kube-scheduler kubelet
systemctl start etcd kube-apiserver kube-controller-manager kube-proxy kube-scheduler kubelet

  SHELL
end

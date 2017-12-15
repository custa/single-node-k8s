# single-node-k8s
单节点 Kubernetes 环境

```
vagrant init centos/7
# 编辑 Vagrantfile，安装 etcd 和 Kubernetes
# 安装 vagrant-vbguest 插件，用于 centos/7 中自动安装 Virtualbox Guest Additions
vagrant plugin install vagrant-vbguest
vagrant up
```



# 1、安装

首先需要安装 VirtualBox，下载地址：<https://www.virtualbox.org/wiki/Downloads/> 。


Mac 上安装 virtualbox：

    brew install virtualbox

安装 Vagrant，下载地址：<http://downloads.vagrantup.com/>。


Mac 上安装：

    brew install vagrant



# 2、下载镜像

Vagrant 提供了很多镜像，你可以去[网站](https://app.vagrantup.com/boxes/search)搜索你想要的镜像（box），然后手动下载。


也可以使用命令下载，例如

    $ vagrant box add generic/centos7

按照提示输入`5` 选择virtualbox：

    ==> box: Loading metadata for box 'generic/centos7'
        box: URL: https://vagrantcloud.com/generic/centos7
    This box can work with multiple providers! The providers that it
    can work with are listed below. Please review the list and choose
    the provider you will be working with.
    1) docker
    2) hyperv
    3) libvirt
    4) parallels
    5) virtualbox
    6) vmware_desktop
    Enter your choice: 5
    ==> box: Adding box 'generic/centos7' (v3.3.2) for provider: virtualbox
        box: Downloading: https://vagrantcloud.com/generic/boxes/centos7/versions/3.3.2/providers/virtualbox.box
        box: Calculating and comparing box checksum...
    ==> box: Successfully added box 'generic/centos7' (v3.3.2) for 'virtualbox'!

镜像都被安装在 `~/.vagrant.d/boxes` 目录下面，可以查看安装了哪些镜像：

    $  vagrant box list
    generic/centos7 (virtualbox, 3.3.2)



# 3、创建虚拟机

创建一个目录，然后初始化一个vagrant虚拟机：

    $ mkdir centos7 && cd centos7
    $ vagrant init generic/centos7

查看生成的文件：

    cat Vagrantfile

去掉注释之后的内容如下：

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    Vagrant.configure("2") do |config|
      config.vm.box = "generic/centos7"
    end



# 4、启动虚拟机

    vagrant up

你会看到终端显示了启动过程，启动完成后，我们就可以用 SSH 登录虚拟机了：

    vagrant ssh



# 5、实战



## 5.1 搭建多节点不同配置

每台机器不同配置：

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  # Vagrant Global Config
  config.vm.box = "generic/centos7"
  config.vm.provision :shell, :path => "bootstrap.sh"
  
  config.vm.define "node1" do |config|
    config.vm.network "private_network", ip: "192.168.56.11"
    #添加一个桥接网络
    config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
    config.vm.hostname = "server"
  end
  
  config.vm.define "node2" do |config|
    v.vm.network "private_network", ip: "192.168.56.12"
    v.vm.hostname = "node1"
  end
  
  config.vm.define "node3" do |config|
    config.vm.network "private_network", ip: "192.168.56.13"
    config.vm.hostname = "node2"
  end
end
```

改为批量配置：

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "generic/centos7"
  config.vm.provision :shell, :path => "bootstrap.sh"
  (1..3).each do |i|
    config.vm.define "node#{i}" do |config|
        config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--name", "node#{i}", 
        "--memory", "1024",'--cpus', 1]
        v.gui = false
      end
      config.vm.network "private_network", ip: "192.168.56.1#{i}"
      config.vm.hostname = "node#{i}"
    end
  end
end
```



## 5.2 配置 SSH 无密码登陆

查看ssh配置：

```bash
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/june/workspace/vagrant/devops/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

> 注意：
>
> - vagrant将PasswordAuthentication设置为no了，这样就无法通过密码登陆，也不能设置root用户无密码登陆到其他节点，需要改成false

scp 拷贝文件到虚拟机：

```bash
scp -i ~/.vagrant/machines/default/virtualbox/private_key test.sql vagrant@192.168.56.100:~
```

参考：<https://stackoverflow.com/questions/10864372/how-to-ssh-to-vagrant-without-actually-running-vagrant-ssh> ，这是SSH登陆：

    ssh $(vagrant ssh-config | awk 'NR>1 {print " -o "$1"="$2}') localhost

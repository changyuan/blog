---
title: vagrant usage
date: 2016-11-10 14:46:53
updated: 2016-11-10 14:46:53
author: change
tags:
categories:
---

### VirtualBox， Vagrant
安装[VirtualBox](https://www.virtualbox.org/wiki/Downloads), 安装[vagrant](https://www.vagrantup.com/downloads.html)

### 安装Box 
你可以把它想成是一个箱子，里面装了一些东西。在用 Vagrant 创建虚拟机的时候，需要用到 Box ，它里面会包装操作系统的镜像，不同的 Box 带的操作系统可能是不一样的，比如 CentOS，Ubuntu 等等，你可以基于它们去创建自己版本的 Box，比如在虚拟机上安装一些软件，然后把它重新打包成 Box。

在 `vagrant添加 Box` ，Vagrant 提供的云服务 ,要把 Box 下载到本地的电脑上，交给 Vagrant 去管理，这样在创建虚拟机的时候，Vagrant 会复制一份你指定的 Box 到你的项目里面，这样你在这个虚拟机上的操作，就不会影响到其它的项目。

<!-- more -->

> http://www.vagrantbox.es/
> https://atlas.hashicorp.com/boxes/search

```bash
vagrant box add {title} {box}    # title 为起的名字，box为在线URL地址，或者离线box路径

```
这里**可以安装很多个box，以服务不同的项目环境**，可以随时通过`vagrant box remove {title}`,所有的命令可以通过vagrant -h 查看。


### 查看已安装的box
`vagrant box list`


### 初始化项目环境
``` bash
# 到项目目录
cd project
# 查看已经装的box
vagrant box list

# 初始化项目，之后回生成Vagrantfile文件，这个是ruby写的配置文件有一些参数设置

`vagrant init {title}`  ，title：为`config.vm.box`重命名，title可略则默认的`config.vm.box=”base”`

# demo
vagrant box add centos6 https://github.com/tommy-muehle/puppet-vagrant-boxes/releases/download/1.0.0/centos-6.6-x86_64.box

```


### 启动 并查看状态
```bash
vagrant up

vagrant status
```

### 控制虚拟机

```bash
# mac
vagrant ssh


# windows 用其他ssh链接工具

 Host：127.0.0.1
 Port：2200
 Username：vagrant
 password：vagrant

```

### 打包分发，生成box供其他人使用

```bash
vagrant package


# 之后别人拿到box，相当于安装了一个box又 ，之后就直接初始化这个
vagrant box add hahaha ~/box/package.box  # 添加 package.box 镜像并命名为 hahaha

cd ~/dev  # 切换到项目目录,之后初始化
vagrant init hahaha  

```
### 集成预安装

你会发现每次都修改了一点点内容，再打包分发给其他用户其实很麻烦，为此 Vagrant 还提供了更为便捷的预安装定制。打开 Vagrantfile 文件末尾处有下面被注释的代码：
```bash
config.vm.provision "shell", inline: <<-SHELL
   apt-get update
   apt-get install -y apache2
SHELL
```
如果你不是初次运行，同时又修改了这里的命令，想让系统再次运行这里面的命令，你可以使用 `vagrant reload --provision` 进行重载。所以在这种情况下，你只要将 Vagrantfile 共享给团队的其他成员就可以了，其他成员运行相同的命令即可。更多[参考](https://www.vagrantup.com/docs/provisioning/)

你还可以把要运行的命令单独写在一个文件里存放在相同的目录下，比如 bootstrap.sh：
```bash
#!/usr/bin/env bash

apt-get update
apt-get install -y apache2
if ! [ -L /var/www ]; then
  rm -rf /var/www
  ln -fs /vagrant /var/www
fi
```
然后在 Vagrantfile 里这样添加：
```bash
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/precise64"
  ...

  config.vm.provision "shell", path: "bootstrap.sh"  # 添加这行
end
```
[更多参考](https://www.vagrantup.com/docs/cli/index.html)


### yii2 的vagrant 的 DEMO  配置
```bash
vagrant box add ubuntu/trusty64
# 如果已经有了vagrantfile文件就不用 vargrant init 了

vagrant up


```

Vagrantfile 配置文件：

```Vagrantfile
require 'yaml'
require 'fileutils'

domains = {
  frontend: 'y2aa-frontend.dev',
  backend:  'y2aa-backend.dev'
}

config = {
  local: './vagrant/config/vagrant-local.yml',
  example: './vagrant/config/vagrant-local.example.yml'
}

# copy config from example if local config not exists
FileUtils.cp config[:example], config[:local] unless File.exist?(config[:local])
# read config
options = YAML.load_file config[:local]

# check github token
if options['github_token'].nil? || options['github_token'].to_s.length != 40
  puts "You must place REAL GitHub token into configuration:\n/yii2-app-advancded/vagrant/config/vagrant-local.yml"
  exit
end

# vagrant configurate
Vagrant.configure(2) do |config|
  # select the box
  config.vm.box = 'ubuntu/trusty64'

  # should we ask about box updates?
  config.vm.box_check_update = options['box_check_update']

  config.vm.provider 'virtualbox' do |vb|
    # machine cpus count
    vb.cpus = options['cpus']
    # machine memory size
    vb.memory = options['memory']
    # machine name (for VirtualBox UI)
    vb.name = options['machine_name']
  end

  # machine name (for vagrant console)
  config.vm.define options['machine_name']

  # machine name (for guest machine console)
  config.vm.hostname = options['machine_name']

  # network settings
  config.vm.network 'private_network', ip: options['ip']

  # sync: folder 'yii2-app-advanced' (host machine) -> folder '/app' (guest machine)
  config.vm.synced_folder './', '/app', owner: 'vagrant', group: 'vagrant'

  # disable folder '/vagrant' (guest machine)
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # hosts settings (host machine)
  config.vm.provision :hostmanager
  config.hostmanager.enabled            = true
  config.hostmanager.manage_host        = true
  config.hostmanager.ignore_private_ip  = false
  config.hostmanager.include_offline    = true
  config.hostmanager.aliases            = domains.values

  # provisioners
  config.vm.provision 'shell', path: './vagrant/provision/once-as-root.sh', args: [options['timezone']]
  config.vm.provision 'shell', path: './vagrant/provision/once-as-vagrant.sh', args: [options['github_token']], privileged: false
  config.vm.provision 'shell', path: './vagrant/provision/always-as-root.sh', run: 'always'

  # post-install message (vagrant console)
  config.vm.post_up_message = "Frontend URL: http://#{domains[:frontend]}\nBackend URL: http://#{domains[:backend]}"
end

```

```once-as-vagrant.sh

#!/usr/bin/env bash

#== Import script args ==

github_token=$(echo "$1")

#== Bash helpers ==

function info {
  echo " "
  echo "--> $1"
  echo " "
}

#== Provision script ==

info "Provision-script user: `whoami`"

info "Configure composer"
composer config --global github-oauth.github.com ${github_token}
echo "Done!"

info "Install plugins for composer"
composer global require "fxp/composer-asset-plugin:~1.1.1" --no-progress

info "Install codeception"
composer global require "codeception/codeception=2.0.*" "codeception/specify=*" "codeception/verify=*" --no-progress
echo 'export PATH=/home/vagrant/.config/composer/vendor/bin:$PATH' | tee -a /home/vagrant/.profile

info "Install project dependencies"
cd /app
composer --no-progress --prefer-dist install

info "Init project"
./init --env=Development --overwrite=y

info "Apply migrations"
./yii migrate <<< "yes"

info "Create bash-alias 'app' for vagrant user"
echo 'alias app="cd /app"' | tee /home/vagrant/.bash_aliases

info "Enabling colorized prompt for guest console"
sed -i "s/#force_color_prompt=yes/force_color_prompt=yes/" /home/vagrant/.bashrc

```


```once-as-root.sh
	#!/usr/bin/env bash

#== Import script args ==

timezone=$(echo "$1")

#== Bash helpers ==

function info {
  echo " "
  echo "--> $1"
  echo " "
}

#== Provision script ==

info "Provision-script user: `whoami`"

info "Allocate swap for MySQL 5.6"
fallocate -l 2048M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab

info "Configure locales"
update-locale LC_ALL="C"
dpkg-reconfigure locales

info "Configure timezone"
echo ${timezone} | tee /etc/timezone
dpkg-reconfigure --frontend noninteractive tzdata

info "Prepare root password for MySQL"
debconf-set-selections <<< "mysql-server-5.6 mysql-server/root_password password \"''\""
debconf-set-selections <<< "mysql-server-5.6 mysql-server/root_password_again password \"''\""
echo "Done!"

info "Update OS software"
apt-get update
apt-get upgrade -y

info "Install additional software"
apt-get install -y git php5-curl php5-cli php5-intl php5-mysqlnd php5-gd php5-fpm nginx mysql-server-5.6

info "Configure MySQL"
sed -i "s/.*bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/my.cnf
echo "Done!"

info "Configure PHP-FPM"
sed -i 's/user = www-data/user = vagrant/g' /etc/php5/fpm/pool.d/www.conf
sed -i 's/group = www-data/group = vagrant/g' /etc/php5/fpm/pool.d/www.conf
sed -i 's/owner = www-data/owner = vagrant/g' /etc/php5/fpm/pool.d/www.conf
echo "Done!"

info "Configure NGINX"
sed -i 's/user www-data/user vagrant/g' /etc/nginx/nginx.conf
echo "Done!"

info "Enabling site configuration"
ln -s /app/vagrant/nginx/app.conf /etc/nginx/sites-enabled/app.conf
echo "Done!"

info "Initailize databases for MySQL"
mysql -uroot <<< "CREATE DATABASE yii2advanced"
mysql -uroot <<< "CREATE DATABASE yii2_advanced_tests"
echo "Done!"

info "Install composer"
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
```

[示例参考](http://www.yiichina.com/tutorial/979)
[示例参考](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/start-installation.md)
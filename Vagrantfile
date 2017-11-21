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
  config.vm.box = "centos74_ssl"

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
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
  end

  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    declare -A HOST_NAMES
    # application name constant start
    # example => HOST_NAMES[`project_name`]=app_host_name
    HOST_NAMES[fuel_test]=local.fueltest
    HOST_NAMES[test_site]=local.testsite
    # application name constant
    # if firewall activated, disable and shutdown firewall
    FIREWALLD_STATUS=`systemctl status firewalld | grep inactive`
    if [ -z "$FIREWALLD_STATUS" ]; then
      echo "---------------------------- stop firewalld -----------------------------------"
      systemctl stop firewalld
    fi
    FIREWALLD_DISABLE=`systemctl status firewalld | grep disabled`
    if [ -z "$FIREWALLD_DISABLE" ]; then
      echo "---------------------------- disable firewalld -----------------------------------"
      systemctl disable firewalld
    fi
    WGET=`rpm -qa wget`
    if [ -z "$WGET" ]; then
      echo "---------------------------- install wget -----------------------------------"
      yum install -y wget
    fi
    VIM=`rpm -qa vim-enhanced`
    if [ -z "$VIM" ]; then
      echo "---------------------------- install vim -----------------------------------"
      yum install -y vim
    fi
    GIT=`rpm -qa git`
    if [ -z "$GIT" ]; then
      echo "---------------------------- install git -----------------------------------"
      yum install -y git
    fi
    NETTOOLS=`rpm -qa net-tools`
    if [ -z "$NETTOOLS" ]; then
      echo "---------------------------- install net-tools -----------------------------------"
      yum install -y net-tools
    fi
    EPEL=`rpm -qa epel-release`
    if [ -z "$EPEL" ]; then
      echo "---------------------------- install epel-repository -----------------------------------"
      yum install -y epel-release
    fi
    REMI=`rpm -qa remi-release`
    if [ -z "$REMI" ]; then
      echo "---------------------------- install remi-repository -----------------------------------"
      rpm -Uvh https://rpms.remirepo.net/enterprise/remi-release-7.rpm
    fi
    NGINX=`rpm -qa nginx`
    if [ -z "$NGINX" ]; then
      echo "---------------------------- install nginx-repository -----------------------------------"
      echo '[nginx]' > /etc/yum.repos.d/nginx.repo
      echo 'name=nginx repo' >> /etc/yum.repos.d/nginx.repo
      echo 'baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/' >> /etc/yum.repos.d/nginx.repo
      echo 'gpgcheck=0' >> /etc/yum.repos.d/nginx.repo
      echo 'enabled=1' >> /etc/yum.repos.d/nginx.repo
    fi
    MYSQLD=`rpm -qa mysql57-community-release`
    if [ -z "$MYSQLD" ]; then
      echo "---------------------------- install mysql-repository -----------------------------------"
      rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
    fi
    yum update -y
    PHP="/bin/php"
    if [ ! -f $PHP ]; then
      echo "---------------------------- install php -----------------------------------"
      yum install -y --enablerepo=remi,remi-php71 php php-devel php-mbstring php-pdo php-gd php-process php-mysqlnd php-pear composer
    fi
    XDEBUG=`php -i | grep xdebug`
    if [ -z $XDEBUG ]; then
      echo "---------------------------- install xdebug -----------------------------------"
      pecl install xdebug
      sed -i -e '1685a [xdebug]\n' /etc/php.ini
      sed -i -e '1686a zend_extension = /usr/lib64/php/modules/xdebug.so' /etc/php.ini
      sed -i -e '1687a xdebug.remote_autostart = 1' /etc/php.ini
      sed -i -e '1688a xdebug.remote_enable = 1' /etc/php.ini
      sed -i -e '1689a xdebug.remote_host = 10.0.2.2' /etc/php.ini
      sed -i -e '1690a xdebug.remote_port = 9001' /etc/php.ini
    fi
    PHPFPM="/usr/sbin/php-fpm"
    if [ ! -f $PHPFPM ]; then
      echo "---------------------------- install php-fpm -----------------------------------"
      yum install -y --enablerepo=remi,remi-php71 php-fpm
      sed -i -e "s/apache/vagrant/g" /etc/php-fpm.d/www.conf
      systemctl enable php-fpm
      systemctl start php-fpm
    fi
    NGINX_EXE="/usr/sbin/nginx"
    if [ ! -f $NGINX_EXE ]; then
      echo "---------------------------- install nginx -----------------------------------"
      yum install -y nginx
      systemctl enable nginx
      systemctl start nginx
    fi
    MYSQLD_EXE="/usr/sbin/mysqld"
    if [ ! -f $MYSQLD_EXE ]; then
      echo "---------------------------- install mysqld -----------------------------------"
      yum install -y mysql-server mysql-client
      systemctl enable mysqld
      systemctl start mysqld
    fi
    NEED_NGINX_RESTART=false
    for key in ${!HOST_NAMES[*]} ; do
      CONF="/etc/nginx/conf.d/$key.conf"
      if [ ! -f $CONF ]; then
      echo "---------------------------- add vhost ${key} -----------------------------------"
        cp /vagrant/default.conf /etc/nginx/conf.d/$key.conf
        chmod 644 /etc/nginx/conf.d/$key.conf
        sed -i -e "s/hostname/${HOST_NAMES[$key]}/g" /etc/nginx/conf.d/$key.conf
        sed -i -e "s/appname/${key}/g" /etc/nginx/conf.d/$key.conf
        echo "127.0.0.1 ${HOST_NAMES[$key]}" >> /etc/hosts
        NEED_NGINX_RESTART=true
      fi
    done
    if $NEED_NGINX_RESTART ; then
      echo "---------------------------- restart nginx -----------------------------------"
      systemctl restart nginx
    fi
    OIL="/usr/local/bin/oil"
    if [ ! -f $OIL ]; then
      echo "---------------------------- oil command install -----------------------------------"
      curl -k https://get.fuelphp.com/oil | sh
    fi
  SHELL
end

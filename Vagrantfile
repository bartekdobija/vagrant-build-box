# OS configuration
$system_config = <<SCRIPT
  # disable IPv6
  if [ "$(grep disable_ipv6 /etc/sysctl.conf | wc -l)" == "0" ]; then
    echo "net.ipv6.conf.all.disable_ipv6=1" >> /etc/sysctl.conf \
      && echo "net.ipv6.conf.default.disable_ipv6=1" >> /etc/sysctl.conf \
      && sysctl -f /etc/sysctl.conf \
      && sysctl -p

    sysctl -w net.ipv6.conf.all.disable_ipv6=1
    sysctl -w net.ipv6.conf.default.disable_ipv6=1

  fi

  # this should be a persistent config
  ulimit -n 65536
  ulimit -s 10240
  ulimit -c unlimited

  systemctl disable firewalld && systemctl stop firewalld
  
  # Add entries to /etc/hosts
  ip=$(ip a s eth1 | awk '/inet/ {split($2, a,"/"); print a[1] }')
  host=$(hostname)
  echo "127.0.0.1 localhost" > /etc/hosts
  echo "$ip $host" >> /etc/hosts
  if [ "$(grep vm.swappiness /etc/sysctl.conf | wc -l)" == "0" ]; then
    echo "vm.swappiness=0" >> /etc/sysctl.conf && sysctl vm.swappiness=0
  fi

  # disable selinux 
  sudo setenforce 0
  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

  # disable swap 
  swapoff -a

SCRIPT

# YUM configuration
$yum_config = <<SCRIPT
  rpm -i https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm 2> /dev/null
  yum -y remove java-1.6* > /dev/null
  yum -y install wget R python-setuptools pandoc curl curl-devel libxml2 libxml2-devel openssl openssl-devel
SCRIPT


# GCC compiler
$devtools_config  = <<SCRIPT
  yum -y groupinstall 'Development Tools' \
    && yum -y install curl cmake snappy snappy-devel openssl openssl-devel pandoc pandoc-citeproc 
SCRIPT

$protobuf_config = <<SCRIPT

  wget -q -P /tmp/ https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz \
    && tar zxf /tmp/protobuf-2.5.0.tar.gz -C /tmp/ \
    && cd /tmp/protobuf-2.5.0 \
    && ./autogen.sh \
    && ./configure \
    && echo "building protobuf" \
    && make > /dev/null \
    && echo "installing protobuf" \
    && make install > /dev/null \
    && ldconfig

SCRIPT

# MVN configuration
$mvn_config = <<SCRIPT
  MVN_LINK=/usr/local/maven
  if [ ! -e ${MVN_LINK} ]; then
    MVN_VERSION=3.6.1
    wget -q -P /usr/local/ http://ftp.heanet.ie/mirrors/www.apache.org/dist/maven/maven-3/${MVN_VERSION}/binaries/apache-maven-${MVN_VERSION}-bin.tar.gz \
      && tar zxf /usr/local/apache-maven-${MVN_VERSION}-bin.tar.gz -C /usr/local \
      && ln -s /usr/local/apache-maven-${MVN_VERSION} ${MVN_LINK}

      echo "export PATH=\\${PATH}:/usr/local/maven/bin" > /etc/profile.d/mvn.sh


  fi
SCRIPT

$information = <<SCRIPT
  ip=$(ip a s eth1 | awk '/inet/ {split($2, a,"/"); print a[1]}')
  echo "Guest IP address: $ip"
  echo "$ip $(hostname)"
SCRIPT

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"
  config.vm.hostname = "build.box.com"
  config.vm.network :private_network, type: "dhcp"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "build-box"
    vb.cpus = 4
    vb.memory = 8192
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
  end

  config.vm.provision :shell, :name => "system_config", :inline => $system_config

  config.vm.provision :shell, :name => "yum_config", :inline => $yum_config
  config.vm.provision :shell, :name => "devtools_config", :inline => $devtools_config
  config.vm.provision :shell, :name => "mvn_config", :inline => $mvn_config
  config.vm.provision :shell, :name => "protobuf_config", :inline => $protobuf_config
  config.vm.provision :shell, :name => "information", :inline => $information

end

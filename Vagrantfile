# OS configuration
$system_config = <<SCRIPT

  # disable IPv6
  if [ "$(grep disable_ipv6 /etc/sysctl.conf | wc -l)" == "0" ]; then
    echo "net.ipv6.conf.all.disable_ipv6=1" >> /etc/sysctl.conf \
      && sysctl -f /etc/sysctl.conf
  fi

  # this should be a persistent config
  ulimit -n 65536
  ulimit -s 10240
  ulimit -c unlimited

  service iptables stop && chkconfig iptables off

  # Add entries to /etc/hosts
  ip=$(ifconfig eth1 | awk -v host=$(hostname) '/inet addr/ {print substr($2,6)}')
  host=$(hostname)
  echo "127.0.0.1 localhost" > /etc/hosts
  echo "$ip $host" >> /etc/hosts

  if [ "$(grep vm.swappiness /etc/sysctl.conf | wc -l)" == "0" ]; then
    echo "vm.swappiness=0" >> /etc/sysctl.conf && sysctl vm.swappiness=0
  fi

SCRIPT

# YUM configuration
$yum_config = <<SCRIPT
  rpm -i http://mirror.unl.edu/epel/6/x86_64/epel-release-6-8.noarch.rpm 2> /dev/null
  yum -y remove java-1.6* > /dev/null
  yum -y install wget R
SCRIPT

# GCC compiler
$devtools_config  = <<SCRIPT
  yum -y groupinstall 'Development Tools' && yum -y install curl
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

$jdk_config = <<SCRIPT
  wget -q --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.rpm" \
    && rpm -i jdk-7u79-linux-x64.rpm

  echo "export JAVA_HOME=/usr/java/default" > /etc/profile.d/java.sh
SCRIPT

# MVN configuration
$mvn_config = <<SCRIPT
  MVN_LINK=/usr/local/maven
  if [ ! -e ${MVN_LINK} ]; then
    MVN_VERSION=3.3.9
    wget -q -P /usr/local/ http://ftp.heanet.ie/mirrors/www.apache.org/dist/maven/maven-3/${MVN_VERSION}/binaries/apache-maven-${MVN_VERSION}-bin.tar.gz \
      && tar zxf /usr/local/apache-maven-${MVN_VERSION}-bin.tar.gz -C /usr/local \
      && ln -s /usr/local/apache-maven-${MVN_VERSION} ${MVN_LINK}

      echo "export PATH=\\${PATH}:/usr/local/maven/bin" > /etc/profile.d/mvn.sh


  fi
SCRIPT

$information = <<SCRIPT
  ip=$(ifconfig eth1 | awk -v host=$(hostname) '/inet addr/ {print substr($2,6)}')
  echo "Guest IP address: $ip"
  echo "$ip build.box.com"
SCRIPT

Vagrant.configure(2) do |config|

  config.vm.box = "boxcutter/centos66"
  config.vm.hostname = "build.box.com"
  config.vm.network :public_network, :mac => "0800DEADBEEF"

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
  config.vm.provision :shell, :name => "jdk_config", :inline => $jdk_config
  config.vm.provision :shell, :name => "mvn_config", :inline => $mvn_config
  config.vm.provision :shell, :name => "protobuf_config", :inline => $protobuf_config
  config.vm.provision :shell, :name => "information", :inline => $information

end

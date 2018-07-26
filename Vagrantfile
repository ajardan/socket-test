# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define 'server', primary: true do |server|
    server.vm.box = "centos/7"
    # Ports to be able to see services from machine
    server.vm.network "forwarded_port", guest: 80, host: 8080
    server.vm.network "forwarded_port", guest: 443, host: 8443
    server.vm.network "forwarded_port", guest: 8083, host: 8083
    server.vm.network "private_network", ip: "192.168.192.1", netmask: "255.255.192.0"
    
    server.vm.provider "virtualbox" do |v|
      v.memory = 512
      v.cpus = 2
    end

    server.vm.provision "shell", inline: <<-SHELL
      yum install -y yum-utils device-mapper-persistent-data lvm2
      yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
      yum install -y docker-ce httpd-tools lsof
      sed -i 's#^ExecStart=/usr/bin/dockerd$#ExecStart=/usr/bin/dockerd --experimental#g' /usr/lib/systemd/system/docker.service
      # some network tuning so we can handle many connections from loader
      sysctl -w net.core.somaxconn=32768
      sysctl -w net.ipv4.ip_local_port_range="2048	65535"
      sysctl -w net.ipv4.tcp_max_syn_backlog=16384
      sysctl -w net.ipv4.tcp_syncookies=0
      # reload to pick up experimental setting
      systemctl daemon-reload
      systemctl start docker
      docker swarm init --advertise-addr 192.168.192.1
      docker deploy -c /vagrant/service.yml app
      # conntrack is enabled after docker starts, so we set this one here
      sysctl -w net.nf_conntrack_max=524288
      # we need this to make VM's see each other for some reason
      echo "* * * * * root /bin/ping -c 59 192.168.192.2 2&>1 >/dev/null" > /etc/cron.d/ping
    SHELL

  end

  config.vm.define 'loader' do |loader|
    loader.vm.box = "centos/7"
    loader.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 2
    end

    loader.vm.network "private_network", ip: "192.168.192.2", netmask: "255.255.192.0"

    loader.vm.provision "shell", inline: <<-SHELL
      yum install -y httpd-tools lsof
      # some tuning so we can open many connections to server
      sysctl -w net.core.somaxconn=32768
      sysctl -w net.ipv4.ip_local_port_range="2048	65535"
      sysctl -w net.ipv4.tcp_max_syn_backlog=16384
      # corrupt 60% of packets, so we emulate bad clients
      tc qdisc add dev eth1 root netem corrupt 60%
      # add IP addresses so we can make requests with different sources
      for ip_a in {192..210};do for ip_b in {3..254}; do ip a add 192.168.$ip_a.$ip_b dev eth1; done; done
      ping -c 5 192.168.192.1
      # run ab from all the added IP addresses
      for ip_a in {192..210}; do echo {3..254} | xargs -d " " -I ip -P 80 ab -B 192.168.$ip_a.ip -qSd -n 20 -kp /etc/redhat-release -c 5 https://192.168.192.1/; done
    SHELL
  end
end  
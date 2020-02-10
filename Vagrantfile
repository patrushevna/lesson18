# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :net => [
		   #{ip: '192.168.150.145', adapter: 1, netmask: "255.255.255.0", virtualbox__intnet: "internet"} ,                  
		   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "inet_rout"}
               ]
  },

  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "inet_rout"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "directors"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "office_hardware"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "wifi"}                  
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.34', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "office_hardware"}
                ]
  },

  :office1Router => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.0.35', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "office_hardware"},
               {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev-office1"},
               {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test_servers-office1"},
               {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managers-office1"},
               {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office_hardware-office1"}
            ]
  },

  :office1Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.2.194', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office_hardware-office1"}
            ]
  },

  :office2Router => {
    :box_name => "centos/7",
    :net => [
              {ip: '192.168.0.36', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "office_hardware"},
              {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev-office2"},
              {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "testservers-office2"},
              {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office_hardware-office2"}
            ]
  }, 
  
  :office2Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.1.194', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office_hardware-office2"}
            ]
  }

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
      
    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        config.vm.provider "virtualbox" do |v|
          v.memory = 256
        end

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
          when "inetRouter"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              sysctl net.ipv4.conf.all.forwarding=1
              iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
              ip route add 192.168.0.0/16 via 192.168.255.2
              SHELL
          when "centralRouter"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo net.ipv4.conf.all.forwarding=1  >> /etc/sysctl.conf
              echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
              echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "192.168.2.0/24 via 192.168.0.35 dev eth3" >> /etc/sysconfig/network-scripts/route-eth3
              echo "192.168.1.0/24 via 192.168.0.36 dev eth3" >> /etc/sysconfig/network-scripts/route-eth3
              systemctl restart network
              SHELL
          when "centralServer"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              systemctl restart network
              SHELL
          when "office1Router"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              systemctl restart network
              SHELL
          when "office1Server"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.2.193" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              systemctl restart network
              SHELL
          when "office2Router"
              box.vm.provision "shell", run: "always", inline: <<-SHELL
                echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
                echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
                echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
                systemctl restart network
                SHELL
          when "office2Server"
            box.vm.provision "shell", run: "always", inline: <<-SHELL
              echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
              echo "GATEWAY=192.168.1.193" >> /etc/sysconfig/network-scripts/ifcfg-eth1
              systemctl restart network
              SHELL
          
        end

      end

  end
  
end

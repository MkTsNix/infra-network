# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/6",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },

  :office1Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.34', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "of1-dev"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "of1-testservers"},
                   {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "of1-managers"},
                   {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "of1-hw"},
                ]
  },
  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.35', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "of2-dev"},
                   {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "of2-testservers"},
                   {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "of2-hw"},
                ]
  },

  :office1Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.2.66', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "of1-testservers"},
                ]
  },

  :office2Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.1.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "of2-testservers"},
                ]
  },


}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

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
            ip r add 192.168.0.0/16 via 192.168.255.2
            SHELL

        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
            sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.255.1
            ip r add 192.168.2.64/26 via 192.168.0.34
            ip r add 192.168.1.128/26 via 192.168.0.35
            systemctl restart network
            SHELL


        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.0.1
            systemctl restart network
            SHELL

        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
            sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.0.33
            systemctl restart network
            SHELL

        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.2.65" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.2.65
            systemctl restart network
            SHELL

        when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
            sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.33" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.0.33
            systemctl restart network
            SHELL

        when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.1.129" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            ip r del default
            ip r add default via 192.168.1.129
            systemctl restart network
            SHELL
        end

      end

  end
  
  
end


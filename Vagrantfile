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
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "central-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "central-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  :office1Router => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.0.3', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "central-net"},
               {ip: '192.168.2.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "local1-net"},
               {ip: '192.168.2.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "testservers1-net"},
               {ip: '192.168.2.129', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "managers1-net"},
               {ip: '192.168.2.193', adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "hardware1-net"},
            ]
  },
  :office1Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.2.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "local1-net"},
               {adapter: 3, auto_config: false, virtualbox__intnet: true},
               {adapter: 4, auto_config: false, virtualbox__intnet: true},
            ]
},
:office2Router => {
    :box_name => "centos/7",
    :net => [
              {ip: '192.168.0.4', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "central-net"},
              {ip: '192.168.1.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "local2-net"},
              {ip: '192.168.1.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "testservers2-net"},
              {ip: '192.168.1.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hardware2-net"},
            ]
  }, 
  :office2Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "local2-net"},
               {adapter: 3, auto_config: false, virtualbox__intnet: true},
               {adapter: 4, auto_config: false, virtualbox__intnet: true},
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
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.255.1
            ip route add 192.168.2.0/24 via 192.168.0.3
            ip route add 192.168.1.0/24 via 192.168.0.4
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.0.1
            ip route add 192.168.2.0/24 via 192.168.0.3
            ip route add 192.168.1.0/24 via 192.168.0.4
            SHELL
        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
            sysctl -p /etc/sysctl.conf
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.0.1
            ip route add 192.168.1.0/24 via 192.168.0.4
            SHELL
        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.2.1
            ip route add 192.168.1.0/24 via 192.168.2.1
            ip route add 192.168.0.0/24 via 192.168.2.1
            SHELL
        when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
            sysctl -p /etc/sysctl.conf
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.0.1
            ip route add 192.168.2.0/24 via 192.168.0.3
            SHELL
        when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network    
            ip route delete default 2>&1 >/dev/null || true
            ip route add default via 192.168.1.1
            ip route add 192.168.2.0/24 via 192.168.1.1
            ip route add 192.168.0.0/24 via 192.168.1.1
            SHELL
        end

      end

  end
  
  
end


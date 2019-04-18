# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
        },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dircentral-net"},
                   {ip: '192.168.1.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "office2router-net"},
                   {ip: '192.168.2.1', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office1router-net"},
                ]
  },
  :office1Router => {
        :box_name => "centos/7",
        :net => [  
                   #office1
                   {ip: '192.168.2.1', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "devof1-net"},
                   {ip: '192.168.2.65', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "testseroff1-net"},
                   {ip: '192.168.2.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "managoff1-net"},
                   {ip: '192.168.2.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "hardwareoff1-net"},
                   #office2
                   {ip: '192.168.1.1', adapter: 6, netmask: "255.255.255.128", virtualbox__intnet: "off2server-net"},
                   #central
                   {ip: '192.168.0.1', adapter: 7, netmask: "255.255.255.240", virtualbox__intnet: "centralserver-net"},
                ]
  },
  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   #office2
                   {ip: '192.168.1.1', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "devof2-net"},
                   {ip: '192.168.1.129', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "testseroff2-net"},
                   {ip: '192.168.1.193', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "hardwareoff2-net"},
                   #office1
                   {ip: '192.168.2.1', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "off1server-net"},
                   #central
                   {ip: '192.168.0.1', adapter: 6, netmask: "255.255.255.240", virtualbox__intnet: "centralserver-net"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "centralserver-net"},
        ]
  },
  :office1Serve => {
        :box_name => "centos/7",
        :net => [
                   { ip: '192.168.2.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "off1server-net" },
        ]
  },
  :office2Server => {
        :box_name => "centos/7",
        :net => [
                   { ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office2server-net" },
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
        ip route add 192.168.0.0/16 via 192.168.255.2
        # echo "192.168.0.0/16 via 192.168.255.2 dev eth2" >> /etc/sysconfig/network-scripts/route-eth2
        # systemctl restart network
        SHELL
        when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1     
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.255.2\nNETMASK=255.255.255.252\nDEVICE=eth2" > /etc/sysconfig/network-scripts/ifcfg-eth2          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.0.1\nNETMASK=255.255.255.240\nDEVICE=eth3" > /etc/sysconfig/network-scripts/ifcfg-eth3            echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.1.1\nNETMASK=255.255.255.128\nDEVICE=eth4" > /etc/sysconfig/network-scripts/ifcfg-eth4          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.1\nNETMASK=255.255.255.192\nDEVICE=eth5" > /etc/sysconfig/network-scripts/ifcfg-eth5        systemctl restart network
        SHELL
        when "office1Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.1\nNETMASK=255.255.255.192\nDEVICE=eth1:0" > /etc/sysconfig/network-scripts/ifcfg-eth1:0

          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.65\nNETMASK=255.255.255.192\nDEVICE=eth1:1" > /etc/sysconfig/network-scripts/ifcfg-eth1:1
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.129\nNETMASK=255.255.255.192\nDEVICE=eth1:2" > /etc/sysconfig/network-scripts/ifcfg-eth1:2
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.193\nNETMASK=255.255.255.192\nDEVICE=eth1:3" > /etc/sysconfig/network-scripts/ifcfg-eth1:3
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.0.1\nNETMASK=255.255.255.240\nDEVICE=eth2" > /etc/sysconfig/network-scripts/ifcfg-eth2
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.1.1\nNETMASK=255.255.255.128\nDEVICE=eth3" > /etc/sysconfig/network-scripts/ifcfg-eth3          systemctl restart network
      SHELL
      when "office2Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.1.1\nNETMASK=255.255.255.128\nDEVICE=eth1:0" > /etc/sysconfig/network-scripts/ifcfg-eth1:0
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.1.129\nNETMASK=255.255.255.192\nDEVICE=eth1:1" > /etc/sysconfig/network-scripts/ifcfg-eth1:1
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.1.193\nNETMASK=255.255.255.192\nDEVICE=eth1:2" > /etc/sysconfig/network-scripts/ifcfg-eth1:2
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.0.1\nNETMASK=255.255.255.240\nDEVICE=eth2" > /etc/sysconfig/network-scripts/ifcfg-eth2
          echo -e "BOOTPROTO=none\nONBOOT=yes\nIPADDR=192.168.2.1\nNETMASK=255.255.255.192\nDEVICE=eth3" > /etc/sysconfig/network-scripts/ifcfg-eth3        
          systemctl restart network
      SHELL
      when "centralServer"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
          echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
      SHELL
      when "office1Server"    
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
      SHELL
      when "office2Server"    
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          #systemctl restart network
          sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
          systemctl restart sshd
      SHELL
    end
  end
end
end

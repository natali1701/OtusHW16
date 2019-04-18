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
                   {ip: '192.168.255.5', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "office1router-net"},
                   {ip: '192.168.255.9', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "office2router-net"},
                   {ip: '192.168.0.1', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "central-net"},
                ]
  },
  :office1Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1router-net"},
                   {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "devof1-net"},
                   {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "testseroff1-net"},
                   {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managoff1-net"},
                   {ip: '192.168.2.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "hardwareoff1-net"},
                ]
  },
  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2router-net"},
                   {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "devof2-net"},
                   {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "testseroff2-net"},
                   {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "hardwareoff2-net"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.46', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "centralserver-net"},
                ]
        	gateway: { ip: "192.168.0.32", connection: "System eth1" },
                dns: { ip: "8.8.8.8", connection: "System eth1" },
  },
  office1Server => {
        net: [
            { ip: '192.168.2.254', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "office1server-net" },
        ],
        gateway: { ip: "192.168.2.192", connection: "System eth1" },
        dns: { ip: "8.8.8.8", connection: "System eth1" },
  },
  :office2Server => {
        net: [
            { ip: '192.168.1.254', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office2server-net" },
        ],
        gateway: { ip: "192.168.1.192", connection: "System eth1" },
        dns: { ip: "8.8.8.8", connection: "System eth1" },
   },
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = "centos/7"
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
        echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.255.3\nIPADDR=192.168.255.1\nNETMASK=255.255.255.252\nNETWORK=192.168.255.0\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
        SHELL
        when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          #echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.255.3\nIPADDR=192.168.255.2\nNETMASK=255.255.255.252\nNETWORK=192.168.255.0\nONBOOT=yes\nGATEWAY=192.168.255.1\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "DEVICE=eth2\nBOOTPROTO=static\nBROADCAST=192.168.255.7\nIPADDR=192.168.255.5\nNETMASK=255.255.255.252\nNETWORK=192.168.255.4\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth2
          echo "DEVICE=eth3\nBOOTPROTO=static\nBROADCAST=192.168.255.11\nIPADDR=192.168.255.9\nNETMASK=255.255.255.252\nNETWORK=192.168.255.8\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth3
          echo "DEVICE=eth4:1\nBOOTPROTO=static\nBROADCAST=192.168.0.15\nIPADDR=192.168.0.1\nNETMASK=255.255.255.240\nNETWORK=192.168.0.0\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth4:1
          echo "DEVICE=eth4:2\nBOOTPROTO=static\nBROADCAST=192.168.0.63\nIPADDR=192.168.0.33\nNETMASK=255.255.255.192\nNETWORK=192.168.0.32\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth4:2
          echo "DEVICE=eth4:3\nBOOTPROTO=static\nBROADCAST=192.168.0.127\nIPADDR=192.168.0.65\nNETMASK=255.255.255.192\nNETWORK=192.168.0.64\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth4:3
          #Routing
          echo "192.168.2.192/26 via 192.168.255.6 dev eth2" > /etc/sysconfig/network-scripts/route-eth2
          echo "192.168.1.192/26 via 192.168.255.10 dev eth3" > /etc/sysconfig/network-scripts/route-eth3
          systemctl restart network
      SHELL
      when "office1Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.255.7\nIPADDR=192.168.255.6\nNETMASK=255.255.255.252\nNETWORK=192.168.255.4\nGATEWAY=192.168.255.5\nONBOOT=yes\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "DEVICE=eth2\nBOOTPROTO=static\nBROADCAST=192.168.2.63\nIPADDR=192.168.2.1\nNETMASK=255.255.255.192\nNETWORK=192.168.2.0\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth2
          echo "DEVICE=eth3\nBOOTPROTO=static\nBROADCAST=192.168.2.127\nIPADDR=192.168.2.65\nNETMASK=255.255.255.192\nNETWORK=192.168.2.64\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth3
          echo "DEVICE=eth4\nBOOTPROTO=static\nBROADCAST=192.168.2.191\nIPADDR=192.168.2.129\nNETMASK=255.255.255.192\nNETWORK=192.168.2.128\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth4
          echo "DEVICE=eth5\nBOOTPROTO=static\nBROADCAST=192.168.2.255\nIPADDR=192.168.2.193\nNETMASK=255.255.255.192\nNETWORK=192.168.2.192\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth5
          systemctl restart network
      SHELL
      when "office2Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.255.11\nIPADDR=192.168.255.10\nNETMASK=255.255.255.252\nNETWORK=192.168.255.8\nGATEWAY=192.168.255.9\nONBOOT=yes\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "DEVICE=eth2\nBOOTPROTO=static\nBROADCAST=192.168.1.63\nIPADDR=192.168.1.1\nNETMASK=255.255.255.128\nNETWORK=192.168.1.0\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth2
          echo "DEVICE=eth3\nBOOTPROTO=static\nBROADCAST=192.168.1.191\nIPADDR=192.168.1.129\nNETMASK=255.255.255.192\nNETWORK=192.168.1.128\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth3
          echo "DEVICE=eth4\nBOOTPROTO=static\nBROADCAST=192.168.1.255\nIPADDR=192.168.1.193\nNETMASK=255.255.255.192\nNETWORK=192.168.1.192\nONBOOT=yes" > /etc/sysconfig/network-scripts/ifcfg-eth4
          systemctl restart network
      SHELL
      when "centralServer"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.0.63\nIPADDR=192.168.0.34\nNETMASK=255.255.255.192\nNETWORK=192.168.0.32\nGATEWAY=192.168.0.33\nONBOOT=yes\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
      SHELL
      when "office1Server"    
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.2.255\nIPADDR=192.168.2.194\nNETMASK=255.255.255.192\nNETWORK=192.168.2.192\nGATEWAY=192.168.2.193\nONBOOT=yes\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
      SHELL
      when "office2Server"    
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "DEVICE=eth1\nBOOTPROTO=static\nBROADCAST=192.168.1.255\nIPADDR=192.168.1.194\nNETMASK=255.255.255.192\nNETWORK=192.168.1.192\nGATEWAY=192.168.1.193\nONBOOT=yes\nDEFROUTE=yes" > /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
      SHELL
    end
  end
end
end

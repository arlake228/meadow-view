# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "meadowview", primary: true, autostart: true do |meadowview|
    # set box to official CentOS 7 image
    meadowview.vm.box = "centos/7"
    # explcitly set shared folder to virtualbox type. If not set will choose rsync 
    # which is just a one-way share that is less useful in this context
    meadowview.vm.synced_folder ".", "/vagrant", type: "virtualbox"
    # Set hostname
    meadowview.vm.hostname = "meadowview"
    
    #increase memory
    meadowview.vm.provider "virtualbox" do |v|
        v.memory = 2048
    end
    
    # Enable IPv4. Cannot be directly before or after line that sets IPv6 address. Looks
    # to be a strange bug where IPv6 and IPv4 mixed-up by vagrant otherwise and one 
    #interface will appear not to have an address. If you look at network-scripts file
    # you will see a mangled result where IPv4 is set for IPv6 or vice versa
    meadowview.vm.network "private_network", ip: "10.5.5.5"
    
    # Enable IPv6. Currently only supports setting via static IP. Address below in the
    # reserved local address range for IPv6
    meadowview.vm.network "private_network", ip: "fdac:218a:75e5:70c9::15"
    
    #Disable selinux
    meadowview.vm.provision "shell", inline: <<-SHELL
        sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config
    SHELL
    #reload VM since selinux requires reboot. Requires `vagrant plugin install vagrant-reload`
    meadowview.vm.provision :reload

    #Install all requirements and perform initial setup
    meadowview.vm.provision "shell", inline: <<-SHELL
        yum install -y epel-release
        yum install -y gcc \
            kernel-devel \
            kernel-headers \
            dkms \
            make \
            bzip2 \
            perl \
            perl-devel \
            docker \
            net-tools \
            wget \
            git\
            npm\
            rpm-build\
            gcc-c++\
            https://dl.grafana.com/oss/release/grafana-6.4.4-1.x86_64.rpm
        
        #Install wizzy
        npm install -g wizzy
        
        ### Update grafana.ini file ###
        #set default grafana theme to light
        sed -i 's/^;default_theme.*/default_theme = light/' /etc/grafana/grafana.ini
        
        #install carpetplot
        /usr/sbin/grafana-cli plugins install petrslavotinek-carpetplot-panel
        
        #Reload systemctl
        systemctl daemon-reload
        
        #Start grafana
        systemctl enable grafana-server
        systemctl start grafana-server
    SHELL
  end
end

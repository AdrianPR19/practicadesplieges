Vagrant.configure("2") do |config|

    config.vm.define "venus" do |venus|
        venus.vm.box = "debian/bookworm64"  
        venus.vm.hostname = "venus.sistema.test"
        venus.vm.network "private_network", ip: "192.168.57.102"  
        venus.vm.provider "virtualbox" do |vb|
            vb.memory = "512"  
            vb.cpus = 1  
        end
    end

    config.vm.define "tierra" do |tierra|
        tierra.vm.box = "debian/bookworm64"  
        tierra.vm.hostname = "tierra.sistema.test"
        tierra.vm.network "private_network", ip: "192.168.57.103" 
        tierra.vm.provider "virtualbox" do |vb|
            vb.memory = "512"  
            vb.cpus = 1  
        end

        tierra.vm.provision "shell", inline: <<-SSHELL
            sudo apt-get update
            sudo apt-get install -y bind9
            sudo cp -v /vagrant/named /etc/default/named
            sudo systemctl restart bind9

            echo 'dnssec-validation yes;' | sudo tee -a /etc/bind/named.conf.options
            echo 'acl "redes_permitidas" { 127.0.0.0/8; 192.168.57.0/24; };' | sudo tee -a /etc/bind/named.conf.options
            
            sudo systemctl restart bind9
        SSHELL
    end
end

  
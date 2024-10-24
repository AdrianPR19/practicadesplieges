Vagrant.configure("2") do |config|

    config.vm.define "venus" do |venus|
        venus.vm.box = "debian/bookworm64"  
        venus.vm.hostname = "venus.sistema.test"
        venus.vm.network "private_network", ip: "192.168.57.102"  
        venus.vm.provider "virtualbox" do |vb|
            vb.memory = "512"  
            vb.cpus = 1  
        end

        
        venus.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y bind9

            
            echo 'zone "sistema.test" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type slave;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.sistema.test";' | sudo tee -a /etc/bind/named.conf.local
            echo '    masters { 192.168.57.103; };' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

            echo 'zone "57.168.192.in-addr.arpa" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type slave;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.192.168.57";' | sudo tee -a /etc/bind/named.conf.local
            echo '    masters { 192.168.57.103; };' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

            
            sudo systemctl restart bind9
        SHELL
    end

    config.vm.define "tierra" do |tierra|
        tierra.vm.box = "debian/bookworm64"  
        tierra.vm.hostname = "tierra.sistema.test"
        tierra.vm.network "private_network", ip: "192.168.57.103" 
        tierra.vm.provider "virtualbox" do |vb|
            vb.memory = "512"  
            vb.cpus = 1  
        end

        tierra.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install -y bind9
            sudo cp -v /vagrant/named /etc/default/named
            sudo systemctl restart bind9

            
            echo 'dnssec-validation yes;' | sudo tee -a /etc/bind/named.conf.options
            echo 'acl "redes_permitidas" { 127.0.0.0/8; 192.168.57.0/24; };' | sudo tee -a /etc/bind/named.conf.options

            
            sudo sed -i '/options {/a \\nallow-recursion { redes_permitidas; };' /etc/bind/named.conf.options
            sudo sed -i '/allow-query { /a \\nallow-query { redes_permitidas; };' /etc/bind/named.conf.options
            sudo sed -i '/allow-query-cache { /a \\nallow-query-cache { redes_permitidas; };' /etc/bind/named.conf.options

            
            sudo systemctl restart bind9
            
            
            echo 'zone "sistema.test" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type master;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.sistema.test";' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

           
            echo 'zone "57.168.192.in-addr.arpa" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type master;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.192.168.57";' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

        
            sudo systemctl restart bind9
        SHELL
    end
end
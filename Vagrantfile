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
            echo '    negative-cache 7200;' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

            echo 'zone "57.168.192.in-addr.arpa" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type slave;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.192.168.57";' | sudo tee -a /etc/bind/named.conf.local
            echo '    masters { 192.168.57.103; };' | sudo tee -a /etc/bind/named.conf.local
            echo '    negative-cache 7200;' | sudo tee -a /etc/bind/named.conf.local
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
            echo '    negative-cache 7200;' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

            echo 'zone "57.168.192.in-addr.arpa" {' | sudo tee -a /etc/bind/named.conf.local
            echo '    type master;' | sudo tee -a /etc/bind/named.conf.local
            echo '    file "/etc/bind/db.192.168.57";' | sudo tee -a /etc/bind/named.conf.local
            echo '    negative-cache 7200;' | sudo tee -a /etc/bind/named.conf.local
            echo '};' | sudo tee -a /etc/bind/named.conf.local

            echo 'options {' | sudo tee /etc/bind/named.conf.options
            echo '    directory "/var/cache/bind";' | sudo tee -a /etc/bind/named.conf.options
            echo '    forwarders {' | sudo tee -a /etc/bind/named.conf.options
            echo '        208.67.222.222;' | sudo tee -a /etc/bind/named.conf.options
            echo '    };' | sudo tee -a /etc/bind/named.conf.options
            echo '    forward only;' | sudo tee -a /etc/bind/named.conf.options
            echo '};' | sudo tee -a /etc/bind/named.conf.options

            echo '; Archivo de zona para sistema.test' | sudo tee /etc/bind/db.sistema.test
            echo '$TTL 604800' | sudo tee -a /etc/bind/db.sistema.test
            echo '@ IN SOA tierra.sistema.test. root.sistema.test. (' | sudo tee -a /etc/bind/db.sistema.test
            echo '    1 ; Serial' | sudo tee -a /etc/bind/db.sistema.test
            echo '    604800 ; Refresh' | sudo tee -a /etc/bind/db.sistema.test
            echo '    86400 ; Retry' | sudo tee -a /etc/bind/db.sistema.test
            echo '    2419200 ; Expire' | sudo tee -a /etc/bind/db.sistema.test
            echo '    604800 ) ; Negative Cache TTL' | sudo tee -a /etc/bind/db.sistema.test
            echo '; Servidores de nombres' | sudo tee -a /etc/bind/db.sistema.test
            echo '@ IN NS tierra.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo '@ IN NS venus.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo '@ IN MX 10 marte.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo '; Registros A' | sudo tee -a /etc/bind/db.sistema.test
            echo 'tierra IN A 192.168.57.103' | sudo tee -a /etc/bind/db.sistema.test
            echo 'venus IN A 192.168.57.102' | sudo tee -a /etc/bind/db.sistema.test
            echo 'marte IN A 192.168.57.104' | sudo tee -a /etc/bind/db.sistema.test
            echo '; Alias (CNAME)' | sudo tee -a /etc/bind/db.sistema.test
            echo 'ns1 IN CNAME tierra.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo 'ns2 IN CNAME venus.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test

            echo 'mail IN CNAME marte.sistema.test' | sudo tee -a /etc/bind/db.sistema.test
            sudo systemctl restart bind9

            sudo systemctl restart bind9
        SHELL
    end
end

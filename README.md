# Practica DNS - Vagrant


## 1.Contenidos del Repositorio

He creado estos archivos:
- `Vagrantfile`: Archivo de configuración para crear las máquinas virtuales.
- `.gitignore`: Archivo que excluye el directorio `.vagrant` y archivos de backup del control de versiones.
- `README.md`: Este archivo con la descripción del proyecto.
- `LICENSE`:

## 2.Especificaciones de la Red

Todas las máquinas pertenecen a la red **192.168.57.0/24**. A continuación se detallan los equipos involucrados y sus configuraciones:

| Equipo   | FQDN                | IP              |
|----------|---------------------|-----------------|
| **Venus**| `venus.sistema.test` | `192.168.57.102`|
| **Tierra**| `tierra.sistema.test` | `192.168.57.103`|

## Requisitos

Para ejecutar este proyecto necesitaremos:

- **Vagrant** (versión recomendada: 2.x.x)
- **VirtualBox** (u otro proveedor de Vagrant compatible)

## Configuración de las Máquinas Virtuales

El `Vagrantfile` crea las máquinas virtuales **venus** y **tierra** utilizando la caja de Debian. Las configuraciones clave son:

- **Venus**: 
  - IP: `192.168.57.102`
  - Sistema operativo: Debian
- **Tierra**: 
  - IP: `192.168.57.103`
  - Sistema operativo: Debian

### Pasos para la creación:

1. Clona el repositorio en tu máquina local:
   ```bash
   git clone https://github.com/AdrianPR19/practicadesplieges
   cd practicadesplieges
   ```

1. Introduce este codigo en el fichero vagrantfile :
```bash
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
    end
  end
  ```

  ## 3.datos del DNS

  1. Activa solamente la escucha del servidor para el protocolo IPv4.
  ---He añadido en la configuracion de bind  `OPTIONS="-u bind -4"`

  2. Establecer la opción dnssec-validation a yes
  ---He ido al apartado de configuracion  `sudo nano /etc/bind/named.conf.options` y he agregado `dnssec-validation yes;`

  3. Los servidores permitirán las consultas recursivas sólo a los ordenadores en la red 127.0.0.0/8 y en la red 192.168.57.0/24, para ello utilizarán la opción de listas de control de acceso o acl.

  
  ---He accedido al directorio `sudo nano /etc/bind/named.conf.options` y he añadido esta acl: 
  
  ```bash
  acl "redes_permitidas" {
    127.0.0.0/8;      // Localhost
    192.168.57.0/24;  // Red local
    };
  ```
  y he agregado este codigo que hpermite las consultas recursivas:

  ```bash
            sudo sed -i '/options {/a \\nallow-recursion { redes_permitidas; };' /etc/bind/named.conf.options
            sudo sed -i '/allow-query { /a \\nallow-query { redes_permitidas; };' /etc/bind/named.conf.options
            sudo sed -i '/allow-query-cache { /a \\nallow-query-cache { redes_permitidas; };' /etc/bind/named.conf.options
  ```
    


```bash
options {
        directory "/var/cache/bind";

        dnssec-validation yes; 
        recursion yes;         
        allow-recursion { "redes_permitidas"; }; 

        auth-nxdomain no;     
        listen-on-v6 { any; }; 
    };
```

## 4.datos del DNS El servidor maestro será tierra.sistema.test y tendrá autoridad sobre la zona directa e inversa
  ---aqui he configurado la maquina tierra agregando la zona directa y inversa 

    ```bash
        echo 'zone "sistema.test" {' | sudo tee -a /etc/bind/named.conf.local
        echo '    type master;' | sudo tee -a /etc/bind/named.conf.local
        echo '    file "/etc/bind/db.sistema.test";' | sudo tee -a /etc/bind/named.conf.local
        echo '};' | sudo tee -a /etc/bind/named.conf.local

           
        echo 'zone "57.168.192.in-addr.arpa" {' | sudo tee -a /etc/bind/named.conf.local
        echo '    type master;' | sudo tee -a /etc/bind/named.conf.local
        echo '    file "/etc/bind/db.192.168.57";' | sudo tee -a /etc/bind/named.conf.local
        echo '};' | sudo tee -a /etc/bind/named.conf.local
 ```   


## 5.El servidor esclavo será venus.sistema.test y tendrá como maestro a tierra.sistema.test.
  ---aqui he configurado la maquina venus para que sea esclavo agregando la zona directa y inversa 
     
  ```bash
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
  ``` 
  
  
##  6. El tiempo en caché de las respuestas negativas de las zonas (directa e inversa) será de dos horas
(se pone en segundos).

Para esto he agregado este codigo: 
```bash 
echo '    negative-cache 7200;' | sudo tee -a /etc/bind/named.conf.local 
```

##  7. Aquellas consultas que reciba el servidor para la que no está autorizado, deberá reenviarlas
(forward) al servidor DNS 208.67.222.222 (OpenDNS).

Para esto he agregado este codigo: 
```bash 
echo 'options {' | sudo tee /etc/bind/named.conf.options
            echo '    directory "/var/cache/bind";' | sudo tee -a /etc/bind/named.conf.options
            echo '    forwarders {' | sudo tee -a /etc/bind/named.conf.options
            echo '        208.67.222.222;' | sudo tee -a /etc/bind/named.conf.options
            echo '    };' | sudo tee -a /etc/bind/named.conf.options
            echo '    forward only;' | sudo tee -a /etc/bind/named.conf.options
            echo '};' | sudo tee -a /etc/bind/named.conf.options 
```

##  8. Se configurarán los siguientes alias:

He agregado este codigo que añade los alias de tierra y venus:
a. ns1.sistema.test. será un alias de tierra.sistema.test.
b. ns2.sistema.test. será un alias de venus.sistema.test..
```bash
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
            echo '; Registros A' | sudo tee -a /etc/bind/db.sistema.test
            echo 'tierra IN A 192.168.57.103' | sudo tee -a /etc/bind/db.sistema.test
            echo 'venus IN A 192.168.57.102' | sudo tee -a /etc/bind/db.sistema.test
            echo '; Alias (CNAME)' | sudo tee -a /etc/bind/db.sistema.test
            echo 'ns1 IN CNAME tierra.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo 'ns2 IN CNAME venus.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
```

##  9. mail.sistema.test. será un alias de marte.sistema.test.
Para esto he usado este codigo:

```bash
echo 'mail IN CNAME marte.sistema.test' | sudo tee -a /etc/bind/db.sistema.test
```

##  10. mail.sistema.test. será un alias de marte.sistema.test.
Para esto he usado este codigo:

```bash
            echo '@ IN MX 10 marte.sistema.test.' | sudo tee -a /etc/bind/db.sistema.test
            echo 'marte IN A 192.168.57.104' | sudo tee -a /etc/bind/db.sistema.test
```


##  4. Comprobación

 Puedes resolver los registros tipo A.

 ```bash
dig @localhost tierra.sistema.test
dig @localhost venus.sistema.test 
 ```
 Comprueba que se pueden resolver de forma inversa sus direcciones IP.
 ```bash
dig -x 192.168.57.103
dig -x 192.168.57.102
dig -x 192.168.57.104
```
Puedes resolver los alias ns1.sistema.test y ns2.sistema.test.
```bash
dig ns1.sistema.test
dig ns2.sistema.test
```
Realiza la consulta para saber los servidores NS de sistema.test. Debes obtener
tierra.sistema.test y venus.sistema.test
```bash
dig sistema.test NS
```
Realiza la consulta para saber los servidores MX de sistema.test
```bash
dig sistema.test MX
```
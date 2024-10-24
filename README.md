# Practica DMS - Vagrant


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

1. Introduce este codigo en el fichero vagrantfile :

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
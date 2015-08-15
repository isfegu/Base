# Esqueleto de un entorno de desarrollo/producción mediante el uso de Vagrant y Ansible

**Requisitos**: Ubuntu 14.04 LTS como sistema operativo guest/host.

Pese que un entorno de este tipo tendríamos _desarrollo_ corriendo en nuestra máquina dentro de un box de vagrant y _producción_ en un servidor externo, en este esqueleto se usan boxes para _desarrollo_ y para _producción_. Ya que el objetivo de éste es el de tener una estructura inicial con la que empezar y modificar según necesidades. Al final del documento se explica cómo utilizar un servidor externo en el entorno de producción.

## PREPARACIÓN DEL ENTORNO
Instalar vagrant, virtualbox y ansible:

``` bash
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get virtualbox
$ sudo apt-get vagrant
$ sudo apt-get ansible
```

## CLONADO DEL PROYECTO
Cada cual clonará el proyecto donde mejor le convenga, para este tutorial será en _~/Projects/Provision_

## CREACIÓN DE CLAVES
Crearemos dos claves, una para desarrollo y otra para producción.

``` bash
$ ssh-keygen -t rsa -C "correo@electronico.com"
```

Ejecutar el anterior comando una vez dándole a la clave el nombre _dev_ y _pro_ la segunda vez.

Tras la creación se deben copiar **ambas claves públicas** (.pub) en el directorio _common/keys_

Estas claves se copiarán automáticamente en cada _box_ una vez sean iniciadas y se ejecute en cada una de ellas (ver archivo _Vagrantfile_) un _vagrant provision_:

## ACTIVAR LAS CLAVES
Para que nuestra máquina _host_ se pueda comunicar con las _boxes_ usando las claves que acabamos de crear debemos:

``` bash
$ ssh-agent bash
$ ssh-add ~/.ssh/dev
$ ssh-add ~/.ssh/pro
```
## INICIAR LAS MÁQUINAS VIRTUALES
``` bash
~/Projects/Provision/dev$ vagrant up
~/Projects/Provision/dev$ vagrant provision
```
``` bash
~/Projects/Provision/pro$ vagrant up
~/Projects/Provision/pro$ vagrant provision
```

## PROBANDO LA COMUNICACIÓN CON ANSIBLE
Para comprobar que ansible puede acceder a las _boxes_ ejecutaremos:

``` bash
~/Projects/Provision/$ ansible web -i provision/environments/dev/dev -m ping
```
Debiéndonos devolver lo siguiente:
```
10.10.10.10.10 | success >> {
    "changed": false, 
    "ping": "pong"
}
```

``` bash
~/Projects/Provision/$ ansible web -i provision/environments/pro/pro -m ping
```
Debiéndonos devolver lo siguiente:
```
10.10.10.10.11 | success >> {
    "changed": false, 
    "ping": "pong"
}
```

Usamos _web_ puesto que está creado un _group_ con ese nombre para especificar los hosts tanto de _dev_ como _pro_. Ver archivos _provision/environments/pro/pro_ y _provision/environments/dev/dev_.

## CONFIGURACIÓN DE LOS SERVIDORES MEDIANTE ANSIBLE

En función de que queramos aprovisionar *dev* o *pro* ejecutaremos:

Para *dev*
``` bash
~/Projects/Provision/$ ansible-playbook -i provision/environments/dev/dev provision/dev.yml
```

Para *pro*
``` bash
~/Projects/Provision/$ ansible-playbook -i provision/environments/pro/pro provision/pro.yml
```
Lo único que hace este esqueleto es asignar el _hostname_ de cada _host_ pero nos da pie a ver el uso de las variables de ansible especificadas en cada una de los _environments_ en:

- _environments/dev/group_vars/web.yml_
- _environments/pro/group_vars/web.yml_

Se puede ver el _task_ que cambia el _hostname_ en:

- _roles/server/tasks/main.yml_

De esta manera tenemos centralizadas las _tasks_ de aprovisionamiento para ambos _environments_ pero se especifican diferentes valores en función de las variables especificadas para cada uno.

## USAR UN SERVIDOR EXTERNO COMO PRODUCCIÓN

Modificamos el archivo _provision/environments/pro/pro_ especificando la IP del servidor externo.

En la rama __external__ se puede ver este cambio reflejado y probado en un _droplet_ de DigitalOcean.
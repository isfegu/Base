# MANUAL DE USO

## PREPARACIÓN DEL ENTORNO
Instalar vagrant, virtualbox y ansible:

$ sudo apt-add-repository ppa:ansible/ansible

$ sudo apt-get update

$ sudo apt-get virtualbox

$ sudo apt-get vagrant

$ sudo apt-get ansible

## CREACIÓN DE CLAVES
Crearemos dos claves una para desarrollo y otra para producción.

$ ssh-keygen -t rsa -C "correo@electronico.com"

Darle como nombre _dev_ a una y _pro_ a la otra.

Tras la creación se deben copiar ambas claves *públicas* (.pub) en common/keys

Estas claves se copiarán automáticamente en cada *box* una vez sean iniciadas y se ejecute un

$ vagrant provision

### Activar las claves

$ ssh-agent bash

$ ssh-add ~/.ssh/dev

$ ssh-add ~/.ssh/pro

## CONFIGURACIÓN DE LOS SERVIDORES MEDIANTE ANSIBLE

En función de que queramos aprovisionar *dev* o *pro* ejecutaremos:

Para *dev*

$ ansible-playbook -i environments/dev/dev provision/dev.yml

Para *pro*

$ ansible-playbook -i environments/pro/pro provision/pro.yml
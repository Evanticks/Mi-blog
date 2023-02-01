---
title: Ansible + Vagrant, configurar y enrutar servidor web.
categories: Servicios de Red e Internet
tags: Servicios de Red e Internet
---


![Descripción de la imagen](/images/ansibleandvagrant.jpg)


Comenzaremos este post viendo como adjuntamos una box https://app.vagrantup.com/boxes/search
buscamos por ejemplo la de debian/ullseye, vamos a nuestra terminal y escribimos lo siguiente:
`vagrant box add debian/bullseye64`

En ese momento se guardará en `~/.vagrant.d/boxes`
Bueno, dicho esto procedemos a crear la carpeta donde vamos a trabajar y hacemos un `vagrant init`, esto hará que se cree un archivo llamado vagrantfile, el cual debe quedar de la siguiente manera:

```
Vagrant.configure("2") do |config|
    config.vm.define :nodo1 do |nodo1|
      nodo1.vm.box = "debian/bullseye64"
      nodo1.vm.hostname= "nodo1"
      nodo1.vm.synced_folder ".", "/vagrant", disabled: true
      nodo1.vm.network :public_network,
        :dev => "br0",
        :mode => "bridge",
        :type => "bridge",
        use_dhcp_assigned_default_route: true
      nodo1.vm.network :private_network,
        :libvirt__network_name => "red1",
        :libvirt__dhcp_enabled => false,
        :ip => "10.0.0.10",
        :libvirt__forward_mode => "veryisolated"
    end
    config.vm.define :nodo2 do |nodo2|
      nodo2.vm.box = "debian/bullseye64"
      nodo2.vm.hostname = "nodo2"
      nodo2.vm.synced_folder ".", "/vagrant", disabled: true
      nodo2.vm.network :private_network,
        :libvirt__network_name => "red1",
        :libvirt__dhcp_enabled => false,
        :ip => "10.0.0.11",
        :libvirt__forward_mode => "veryisolated",
        use_dhcp_assigned_default_route: true
    end
end

```

Vamos a explicar paso por paso qué vemos en el código:
- El vagrantfile debe estar perfectamente alineado, los end con sus respectivos config.
- Al ser dos máquinas las vamos a llamar nodo1 y nodo2, que una será el router y la segunda el servidor que aloje apache
- Si nos fijamos con atención podemos ver que **nodo1** tiene dos redes, una será pública, la cual tiene un bridge por donde saldrá a internet, y una red privada muy aislada, la cual no contendrá nada más que una ip estática, pondremos que la ruta por defecto va a ser el bridge que nos llevará a internet.
- En **nodo2** tendremos una red privada, más tarde procederemos a ejecutar unas reglas que nos permitan salir por el exterior a través de la red muy aislada con **nat**
  
Una vez hecho esto procedemos a ver de cerca lo que contiene Ansible, el cual ejecutaremos tras iniciar las máquinas de Vagrant junto con sus redes.

![Descripción de la imagen](/images/Ansible.png)

Como podemos ver, hay una serie de carpetas llamadas roles, cada rol va a ejecutar una o varias tareas dependiendo de lo que se trate, por lo tanto así quedaría por ejemplo el activado del bit de forwarding del router:

```
- ansible.posix.sysctl:
    name: net.ipv4.ip_forward
    value: '1'
    sysctl_set: yes
```

Estas tareas tienen un orden, y se realiza dentro de site.yml:
```
- hosts: nodo1
  become: true
  roles:
   - role: copy

- hosts: all
  become: true
  roles:
   - role: commons

- hosts: nodo2
  become: true
  roles:
   - role: copy2
   - role: apache2

- hosts: nodo1
  become: true
  roles:
   - role: ipv4

```

Con un poco de conocimientos en sistemas podemos ver claramente qué realiza cada rol.

Ahora vamos a poner la lupa en `roles/commons/handlers/main.yaml`, una parte muy importante de la ejecución de Ansible:
```
- name: reiniciando maquina
  ansible.builtin.reboot:
    msg: "reboot by Ansible"
    pre_reboot_delay: 5
    post_reboot_delay: 10
    test_command: "whoami"
```

Este handler se ejecuta en el momento en el que se hace un llamamiento, en nuestro caso es cuando copiamos al router un fichero configurado para etc/network/interfaces, con sus interfaces, ruta por defecto e iptables:

![Descripción de la imagen](/images/interfaces.png)

podemos ver como en `roles/copy/tasks/main.yaml` hace el llamamiento al **handler** en el notify:

```
- name: Copiando al etc/network/interfaces
  ansible.builtin.copy:
    src: interfaz_nodo1/interfaces
    dest: /etc/network/
    owner: root
    group: root
    mode: u-rw,g-wx,o-rwx
  notify:
    - reiniciando maquina

```

A continuación podemos copiar nuestra clave pública a través de la tarea **commons** que será lo que nos permita atravesar por ssh sin utilizar la clave vagrant, es decir, la nuestra.

```
- name: Ensure system is updated
  apt: update_cache=yes upgrade=yes

- name: Set authorized key took from file
  authorized_key:
    user: vagrant
    state: present
    key: "{{ lookup('file', '/home/antonio/.ssh/id_rsa.pub') }}"

```

Luego pasamos a la instalación de Apache,a través del módulo apt para instalar apache2, ports.conf es la configuración de los puertos por los que escucha, también alberga el fichero index.html y una plantilla jinja2.

Por último pero no menos importante está el fichero hosts, el cual será el inventario por el cual el sistema sabrá cuál será nodo1 y cuál nodo2, a través de las correspondientes ips que asignaremos,el usuario más la clave privada que asignará vagrant:

```
all:
  children:
    servidores_web:
      hosts:
        nodo1:
          ansible_ssh_host: 192.168.121.212
          ansible_ssh_user: vagrant
          ansible_ssh_private_key_file: ../.vagrant/machines/nodo1/libvirt/private_key
        nodo2:
          ansible_ssh_host: 192.168.121.111
          ansible_ssh_user: vagrant
          ansible_ssh_private_key_file: ../.vagrant/machines/nodo2/libvirt/private_key

```

con esto solo necesitaremos ejecutar el `ansible-playbook site.yaml` y tendremos enrutado y configurado nuestro servidor web.
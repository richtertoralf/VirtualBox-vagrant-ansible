# VirtualBox-vagrant-ansible
Virtuelle Maschinen in VirtualBox mit Vagrant und Ansible erstellen und verwalten .... Erste Schritte

>Manchmal benötige ich ganz schnell eine Testumgebung, z.B. ein virtuelles Netzwerk mit mehreren Computern. Dazu bietet sich VirtualBox und Vagrant an.

## Was machen Vagrant und Ansible?
- Ansible ist ein Automatisierungstool. Es ermöglicht dir, Aufgaben auf mehreren Systemen gleichzeitig auszuführen, ohne dass du dich auf einem einzelnen System anmelden musst. Ansible ist ein mächtiges Werkzeug, das für eine Vielzahl von Aufgaben verwendet werden kann, z. B. für das Provisionieren von virtuellen Maschinen, das Installieren von Software und das Konfigurieren von Systemen.
- Vagrant ist ein Tool zum Erstellen und Verwalten von virtuellen Maschinen. Es ermöglicht dir, virtuelle Maschinen auf deinem Computer zu erstellen und zu verwalten. Vagrant ist ein einfaches und flexibles Werkzeug, das sowohl für Anfänger als auch erfahrene Benutzer geeignet ist.

## Vagrant-Kochrezept: Testumgebung vorbereiten und erstellen
- Computer mit frischem Ubuntu 20.04 LTS als Betriebssystem
```
# System aktualisieren
sudo apt update && sudo apt upgrade -y
# einen schönen Editor installieren
sudo snap install --classic code
# VirtualBox installieren
sudo apt install virtualbox
# Vagrant installieren
sudo apt install vagrant
# ins eigene Home-Verzeichnis wechseln
cd
# einen Ordner für Vagrant erstellen und dorthin wechseln
mkdir vagrant
cd vagrant
# jetzt mein fertiges Vagrantfile von Github downloaden
wget -L https://raw.githubusercontent.com/richtertoralf/VirtualBox-vagrant-ansible/main/Vagrantfile
# mit Vagrant die im Vagrantfile beschriebenen virtuellen Maschinen erstellen lassen
vagrant up
```
Nicht immer funktionert die Netzwerkkonfiguration und Installation auf diese Weise. Manchmal ist im BIOS/UEFI die Virtualisierung noch einzuschalten, manchmal gibt es undursichtige Abhängigkeiten zwischen dem Betriebssystem, der Version von VirtualBox und Vagrant oder finstere Mächte haben sich gegen dich verschworen. Frag einfach ChatGPT oder Google Bard oder einen guten Freund. (Aktuell funktioniert mein Vagrantfile auf einem Windows 11 Host und Oracle VirtualBox Version 7.0.10.)

**Mir geht es so, das ich nach paar Monaten Vieles wieder vergessen habe, wenn ich zwischenzeitlich an anderen Aufgaben gearbeitet habe. Deshalb hier einen Erklärung zur Testumgebung, die ich mir mit dem Vagrantfile erschaffe:**

## Infos zur Testumgebung / Netzwerk
### Die Debian-Router-VM
Die Debian-Router-VM ist die zentrale Schnittstelle zwischen dem internen Netzwerk und dem Host-Netzwerk. Sie bekommt von Vagrant drei Netzwerkkarten:  
- eth0: Diese Karte ist für den Zugriff vom Host-Netzwerk aus zuständig. Sie wird automatisch vom VirtualBox-Host eingerichtet und als "NAT-NIC" verwendet. Dadurch funktioniert der Aufruf `vagrant ssh debian-router`  
- eth1: Diese Karte ist das Gateway für das interne Netzwerk. Sie bekommt die IP-Adresse 192.168.100.1    
- eth2: Diese Karte ist für den Zugriff nach außen zuständig, also der WAN-Port des Routers. Sie bekommt ihre Adresse von einem DHCP-Router, z.B. einer FritzBox aus dem Home-Netz.

Die Debian-Router-VM führt Aufgaben aus, um den Netzwerkverkehr zwischen dem internen Netzwerk und dem Host-Netzwerk zu ermöglichen:  
- IP-Forwarding: IP-Forwarding ist eine Funktion, die es einer VM ermöglicht, Datenverkehr zwischen zwei Netzwerken weiterzuleiten. In diesem Fall wird IP-Forwarding verwendet, um Datenverkehr von den Ubuntu-VMs über die Debian-Router-VM ins Host-Netzwerk zu leiten. Darüber ist auch der Zugriff ins Internet möglich. 
- NAT: NAT (Network Address Translation) ist eine Funktion, die die IP-Adressen von Geräten in einem internen Netzwerk in andere IP-Adressen übersetzt. In diesem Fall wird NAT verwendet, um die IP-Adressen der Ubuntu-VMs in IP-Adressen des Host-Netzwerks zu übersetzen.  

### Die Ubuntu-VMs
Die Ubuntu-VMs haben jeweils zwei Netzwerkkarten:  
- enp0s3: Diese Karte wird wieder als NAT-NIC automatisch von Vagrant eingerichtet.  
- enp0s8: Diese Karte ist für das interne Netzwerk zuständig. Sie wird mit einer IP-Adresse aus dem Bereich 192.168.100.101 - 192.168.100.103 konfiguriert.  
Die Ubuntu-VMs können über die IP-Adressen 192.168.100.101, 192.168.100.102 und 192.168.100.103 über das interne Netzwerk miteinander kommunizieren. Sie können auch über die den Debian-Routers auf das Internet zugreifen.

## Zugriff auf die VM´s vom Host aus via vagrant
`vagrant ssh <hostname>`  z.B. `vagrant ssh debian-router`  

Damit das funktioniert gibt es bei VirtualBox Folgende zu beachten:  
### NAT-Anforderung als erste Netzwerkschnittstelle
Bei VirtualBox erfordert Vagrant, dass das erste an die virtuelle Maschine angeschlossene Netzwerkgerät ein NAT-Gerät ist. Das NAT-Gerät wird für die Portweiterleitung verwendet, wodurch Vagrant SSH-Zugriff auf die virtuelle Maschine erhält.  
Daher werden alle Host-Only- oder Bridged-Netzwerke oder Interne Netzwerke als zusätzliche Netzwerkgeräte hinzugefügt und der virtuellen Maschine als „eth1“, „eth2“ usw. angezeigt. „eth0“ oder „en0“ ist grundsätzlich immer das NAT-Gerät.  
Es ist derzeit nicht möglich, diese Anforderung außer Kraft zu setzen, aber es ist wichtig zu verstehen, dass sie vorhanden ist. Mir hat das etwas Kopfzerbrechen bereitet, da mir diese Regel nicht bekannt war. Man kann zwar im Nachhinein die erste NIC´s löschen, muss dann aber auch das Routing auf den VM´s anpassen. Danach kann ich auch nicht mehr per `vagrant ssh ...` auf die Maschinen zugreifen, kann aber das "normale SSH", z.B.. `ssh user@hostname` verwenden. Für Netzwerksimulationen ist diese Variante der Erschaffung einer Laborumgebung für mich nicht optimal. Vielleicht fällt mir aber auch noch was ein... 

## Geht das auch mit Ansible?
Habe mal Bard gefragt. Er schlägt mit das folgende Playbook dazu vor. Da mir die besondere NAT-Anforderung von Vagrant bei VirtualBox nicht gefällt, werde ich mir Ansible mal genauer anschauen ;-)
```
---
- hosts: all
  vars:
    vm_settings:
      num_vms: 3
      memory: 1024
      private_network_name: "intnet01"
      base_ip: "192.168.100.101"
      box_name: "ubuntu/focal64"
      user_name: "tori"
      user_password: "linux"
      hostname_prefix: "ubuntu"

    debian_router_ip: "192.168.100.1"

  tasks:
  - name: Create Debian router VM
    vm:
      name: "debian-router"
      box: "debian/buster64"
      hostname: "debian-router"
      memory: "{{ vm_settings.memory }}"
      networks:
        - name: "private_network"
          type: "static"
          ip: "{{ debian_router_ip }}"
          virtualbox__intnet: true
        - name: "public_network"
          type: "dhcp"

  - name: Create Ubuntu VMs
    block:
    - name: Create Ubuntu VM
      vm:
        name: "{{ item.vm_name }}"
        box: "{{ vm_settings.box_name }}"
        hostname: "{{ item.vm_name }}"
        memory: "{{ vm_settings.memory }}"
        networks:
          - name: "private_network"
            type: "static"
            ip: "{{ item.vm_ip }}"
            gateways: ["{{ debian_router_ip }}"]
            virtualbox__intnet: true

    - name: Configure user and sudo permissions
      shell: |
        # Install openssh-server
        sudo apt install openssh-server

        # Create user and add to sudo group
        useradd -m -s /bin/bash -g sudo {{ vm_settings.user_name }}

        # Set password
        echo "{{ vm_settings.user_name }}:{{ vm_settings.user_password }}" | chpasswd

        # Enable SSH password authentication
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
        service ssh restart

    loop: "{{ range(vm_settings.num_vms) | map(attribute='number') }}"

  - name: Configure routing on Debian router
    shell: |
      # Enable IP forwarding
      echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
      sysctl -p /etc/sysctl.conf

      # Add NAT routing rule
      sudo apt install iptables

      # Allow all traffic between eth1 and eth2
      sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT
      sudo iptables -A FORWARD -i eth2 -o eth1 -j ACCEPT

      # Enable NAT for eth1
      sudo iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE

  - name: Configure Ubuntu VMs
    block:
    - name: Configure IP address
      shell: |
        sudo ip addr add {{ item.vm_ip }}/24 dev enp0s8

    - name: Configure gateway
      shell: |
        sudo ip route add default via {{ debian_router_ip }} dev enp0s8

    loop: "{{ range(vm_settings.num_vms) | map(attribute='number') }}"
```

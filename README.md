# VirtualBox-vagrant-ansible
Virtuelle Maschinen in VirtualBox mit Vagrant und Ansible erstellen und verwalten .... Erste Schritte

>Manchmal benötige ich ganz schnell eine Testumgebung, z.B. ein virtuelles Netzwerk mit mehreren Computern. Dazu bietet sich VirtualBox und Vagrant an.

## Kochrezept: Testumgebung vorbereiten und erstellen
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

**Mir geht es so, das ich nach paar Monaten, wenn ich andere Aufgabe löse, Vieles wieder vergessen habe. Deshalb hier einen Erklärung zur Testumgebung, die ich mir mit dem Vagrantfile erschaffe:**

## Infos zur Testumgebung
### Die Debian-Router-VM
Die Debian-Router-VM ist die zentrale Schnittstelle zwischen dem internen Netzwerk und dem Host-Netzwerk. Sie bekommt von Vagrant drei Netzwerkkarten:  
- eth0: Diese Karte ist für den Zugriff vom Host-Netzwerk aus zuständig. Sie wird automatisch vom VirtualBox-Host eingerichtet und als "NAT-NIC" verwendet. Dadurch funktioniert der Aufruf `vagrant ssh debian-router`  
- eth1: Diese Karte ist für das interne Netzwerk zuständig. Sie wird mit der IP-Adresse 192.168.100.1 konfiguriert.  
- eth2: Diese Karte ist für den Zugriff von außen zuständig. Sie wird mit einer öffentlichen IP-Adresse konfiguriert.  

Die Debian-Router-VM führt auch einige Aufgaben aus, um den Netzwerkverkehr zwischen dem internen Netzwerk und dem Host-Netzwerk zu ermöglichen:  
- IP-Forwarding: IP-Forwarding ist eine Funktion, die es einer VM ermöglicht, Datenverkehr zwischen zwei Netzwerken weiterzuleiten. In diesem Fall wird IP-Forwarding verwendet, um Datenverkehr von den Ubuntu-VMs über die Debian-Router-VM ins Host-Netzwerk zu leiten.  
- NAT: NAT (Network Address Translation) ist eine Funktion, die die IP-Adressen von Geräten in einem internen Netzwerk in andere IP-Adressen übersetzt. In diesem Fall wird NAT verwendet, um die IP-Adressen der Ubuntu-VMs in IP-Adressen des Host-Netzwerks zu übersetzen.  

### Die Ubuntu-VMs
Die Ubuntu-VMs haben nur zwei Netzwerkkarten:  
- enps03: Diese Karte wird wieder als NAT-NIC automatisch von Vagrant eingerichtet.  
- enps08: Diese Karte ist für das interne Netzwerk zuständig. Sie wird mit einer IP-Adresse aus dem Bereich 192.168.100.101 - 192.168.100.103 konfiguriert.  
Die Ubuntu-VMs können über die IP-Adressen 192.168.100.101, 192.168.100.102 und 192.168.100.103 über das interne Netzwerk miteinander kommunizieren. Sie können auch über die den Debian-Routers auf das Internet zugreifen.

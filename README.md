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
Nicht immer funktionert die Netzwwerkkonfiguration und Installation auf diese Weise. Manchmal ist im BIOS/UEFI die Virtualisierung noch einzuschalten, manchmal gibt es undursichtige Abhängigkeiten zwischen dem Betriebssystem, der Version von VirtualBox und Vagrant oder findstere Mächte haben sich gegen dich verschworen. Frag einfach ChatGPT oder Google Bard oder ;-) 


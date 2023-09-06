Vagrant.configure("2") do |config|

  # Definiere die `vm_settings`-Variable
  vm_settings = {
    num_vms: 3,
    memory: 1024, # 1 GB RAM
    base_ip: "192.168.100.10", # Basis-IP-Adresse für Ubuntu VMs
    box_name: "ubuntu/focal64", # Verwende das Image für Ubuntu 22.04 LTS
    user_name: "tori",
    user_password: "linux",
    hostname_prefix: "ubuntu",
  }

  # Definiere die `debian_router_ip`-Variable
  debian_router_ip = "192.168.100.1"

  # Konfiguration für die Debian-Router-VM
  config.vm.define "debian-router" do |debian|
    debian.vm.box = "debian/buster64"
    debian.vm.hostname = "debian-router"
    debian.vm.network "private_network", type: "static", ip: debian_router_ip

    # Konfiguriere die zweite Netzwerkkarte (Host-Netzwerk)
    debian.vm.network "public_network", type: "dhcp"

    debian.vm.provider "virtualbox" do |vb|
      vb.name = "debian-router"
      vb.memory = vm_settings[:memory]
    end

    # Konfiguriere den neuen Benutzer und weise ihm Sudo-Rechte zu
    debian.vm.provision "shell", inline: <<-SHELL
      # Benutzer erstellen
      useradd -m -s /bin/bash -g sudo #{vm_settings[:user_name]}
  
      # Passwort festlegen
      echo "#{vm_settings[:user_name]}:#{vm_settings[:user_password]}" | chpasswd
  
      # SSH-Passwort-Authentifizierung aktivieren
      sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      service ssh restart
  
      # Aktiviere IP-Forwarding für das Routing
      echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
      sysctl -p /etc/sysctl.conf
  
      # Füge eine NAT-Routing-Regel hinzu, um den Datenverkehr von den Ubuntu-VMs über die Debian-VM ins Host-Netzwerk zu leiten
      sudo apt install iptables
      # Erlaube den gesamten Datenverkehr zwischen eth1 (internes Netzwerk / 192.168.100.0/24) und eth2 (Heimnetzwerk / DHCP)
      sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT
      sudo iptables -A FORWARD -i eth2 -o eth1 -j ACCEPT
      # Aktiviere Network Address Translation (NAT) für das interne Netzwerk (eth1)
      sudo iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE
      # Das wird so nicht immer funktionieren, da die Namen der Schnittstellen abweichen können!!
    SHELL
  end

  # Schleife, um Ubuntu-VMs zu erstellen
  (1..vm_settings[:num_vms]).each do |i|
    vm_name = "#{vm_settings[:hostname_prefix]}#{i}"
    vm_ip = "#{vm_settings[:base_ip]}#{i}"

    config.vm.define vm_name do |node|
      node.vm.box = vm_settings[:box_name]
      node.vm.hostname = vm_name
      node.vm.network "private_network", type: "static", ip: vm_ip, extra: "option routers #{debian_router_ip};"
      node.vm.provider "virtualbox" do |vb|
        vb.name = vm_name
        vb.memory = vm_settings[:memory]
      end

      # Konfiguriere den neuen Benutzer und weise ihm Sudo-Rechte zu
      node.vm.provision "shell", inline: <<-SHELL
        
        # openssh-server installieren
        sudo apt install openssh-server

        # Benutzer erstellen und der Gruppe sudo zuordnen
        useradd -m -s /bin/bash -g sudo #{vm_settings[:user_name]}
  
        # Passwort festlegen
        echo "#{vm_settings[:user_name]}:#{vm_settings[:user_password]}" | chpasswd
  
        # SSH-Passwort-Authentifizierung aktivieren
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
        sudo service ssh restart

      SHELL
    end
  end
end

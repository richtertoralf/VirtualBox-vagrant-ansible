# Definiere die gewünschten Einstellungen
vm_settings = {
  num_vms: 3,
  memory: 1024, # 1 GB RAM
  base_ip: "192.168.100.10", # Basis-IP-Adresse für Ubuntu VMs
  box_name: "ubuntu/focal64", # Verwende das Image für Ubuntu 22.04 LTS
  user_name: "tori",
  user_password: "linux",
  hostname_prefix: "ubuntu",
}

# Benutzername und Passwort für den Debian-Router
debian_user_name = "tori"
debian_user_password = "linux"

# IP-Adresse für den Debian-Router
debian_router_ip = "192.168.100.1"

Vagrant.configure("2") do |config|
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
        # Benutzer erstellen
        useradd -m -s /bin/bash #{vm_settings[:user_name]}
        # Passwort festlegen
        echo "#{vm_settings[:user_name]}:#{vm_settings[:user_password]}" | chpasswd
        # Sudo-Rechte zuweisen
        usermod -aG sudo #{vm_settings[:user_name]}
        
        # SSH-Passwort-Authentifizierung aktivieren
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
        service ssh restart
      SHELL
    end
  end

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
      useradd -m -s /bin/bash #{debian_user_name}
      # Passwort festlegen
      echo "#{debian_user_name}:#{debian_user_password}" | chpasswd
      # Sudo-Rechte zuweisen
      usermod -aG sudo #{debian_user_name}
      
      # SSH-Passwort-Authentifizierung aktivieren
      sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
      service ssh restart
      
      # Aktiviere IP-Forwarding für das Routing
      echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
      sysctl -p /etc/sysctl.conf
      
      # Füge eine NAT-Routing-Regel hinzu, um den Datenverkehr von den Ubuntu-VMs über die Debian-VM ins Host-Netzwerk zu leiten
      sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
    SHELL
  end
end

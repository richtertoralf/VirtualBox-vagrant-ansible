Vagrant.configure("2") do |config|

  # Definiere ein privates Netzwerk
  config.vm.network "private_network", ip: "192.168.69.1", virtualbox__intnet: "fg01.local"

  # Definiere zwei VMs
  vm_desc = [["fg01Ubuntu01", "192.168.69.101"], ["fg01Ubuntu02", "192.168.69.102"]]
  vm_desc.each do |nam, add|

    # Definiere eine VM
    config.vm.define "#{nam}" do |node|

      # Setze das Betriebssystem
      node.vm.box = "ubuntu/jammy64"

      # Setze die IP-Adresse
      node.vm.network "private_network", ip: add

      # Setze den Hostnamen
      node.vm.hostname = "#{nam}.fg01.local"

      # Setze die Speichergröße
      node.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
      end

      # Führe nach dem Start einen Befehl aus
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        useradd -m -s /bin/bash tori
        sudo echo "tori:linux:" | chpasswd
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        sudo systemctl restart sshd
      SHELL

    end

  end

  # Definiere eine Debian VM
  config.vm.define "fg01DebianRouter" do |node|

    # Setze das Betriebssystem
    node.vm.box = "debian/bullseye64"

    # Setze die IP-Adresse
    node.vm.network "private_network", ip: "192.168.95.1"

    # Setze den Hostnamen
    node.vm.hostname = "fg01DebianRouter"

    # Setze die Speichergröße
    node.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end

    # Installiere die erforderlichen Pakete
    node.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y iptables iproute2
    SHELL

    # Konfiguriere den Router
    node.vm.provision "shell", inline: <<-SHELL
      echo "192.168.69.1/24 via 192.168.95.1 dev eth0" >> /etc/iptables/rules.v4
      echo "0.0.0.0/0 via 192.168.95.1 dev eth0" >> /etc/iproute2/rt_tables
      ip route add default via 192.168.95.1 dev eth0
    SHELL

  end

end

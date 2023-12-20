# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  # SSH Configuration
  config.ssh.insert_key = false

  # Ansible Controller
  config.vm.define "ansible-controller" do |controller|
    controller.vm.hostname = "ansible-controller"
    controller.vm.network "public_network", bridge: "Intel(R) Wi-Fi 6 AX201 160MHz"
    controller.vm.provider "virtualbox" do |vb|
      vb.name = "ansible-controller"
      vb.cpus = 2
      vb.memory = "2048"
    end

    # Provisioning for the controller machine
    controller.vm.provision "file", source: "~/.ssh/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
    controller.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"

    # Install Ansible
    controller.vm.provision "shell", inline: <<-SHELL
      sudo apt update -y
      sudo apt-get install gnupg -y
      UBUNTU_CODENAME=jammy
      wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
      echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
      sudo apt update && sudo apt install ansible -y
    SHELL
  end

  # Debian Targets
  (1..2).each do |i|
    config.vm.define "ansible-target-#{i}" do |node|
      node.vm.hostname = "ansible-target-#{i}"
      node.vm.network "public_network", bridge: "Intel(R) Wi-Fi 6 AX201 160MHz"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "ansible-target-#{i}"
        vb.cpus = 1
        vb.memory = "1024"
      end

      # Provisioning for each target machine
      node.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/authorized_keys"

      # Set permissions
      node.vm.provision "shell", inline: "chmod 700 /home/vagrant/.ssh && chmod 600 /home/vagrant/.ssh/authorized_keys"
    end
  end
end

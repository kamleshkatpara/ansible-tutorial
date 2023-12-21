# -*- mode: ruby -*-
# vi: set ft=ruby:

# Define variables for repeatable values
ansible_controller_cpus = 2
ansible_controller_memory = 2048
target_cpus = 1
target_memory = 1024

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false

  # Ansible Controller Configuration
  config.vm.define "ansible-controller" do |controller|
    controller.vm.hostname = "ansible-controller"
    controller.vm.network "public_network", bridge: "Intel(R) Wi-Fi 6 AX201 160MHz"
    
    # VirtualBox Provider Configuration
    controller.vm.provider "virtualbox" do |vb|
      vb.name = "ansible-controller"
      vb.cpus = ansible_controller_cpus
      vb.memory = ansible_controller_memory
    end

    # Provisioning for the Ansible Controller
    controller.vm.provision "file", source: "~/.ssh/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
    controller.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"

    # Install Ansible on the Controller
    controller.vm.provision "shell", inline: <<-SHELL
      sudo apt update -y
      sudo apt-get install gnupg -y
      UBUNTU_CODENAME=jammy
      wget -O - "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg
      echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list
      sudo apt update && sudo apt install ansible -y
      sudo chmod 600 /home/vagrant/.ssh/id_rsa
      echo "Provisioning script completed on ansible-controller"
    SHELL
  end

  # Debian Targets Configuration
  (1..2).each do |i|
    config.vm.define "ansible-target-#{i}" do |node|
      node.vm.hostname = "ansible-target-#{i}"
      node.vm.network "public_network", bridge: "Intel(R) Wi-Fi 6 AX201 160MHz"
      
      # VirtualBox Provider Configuration
      node.vm.provider "virtualbox" do |vb|
        vb.name = "ansible-target-#{i}"
        vb.cpus = target_cpus
        vb.memory = target_memory
      end

      # Provisioning for each Target Machine
      node.vm.provision "file", source: "~/.ssh/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
      node.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"

      # Configure SSH on Target Machines
      node.vm.provision "shell", inline: <<-SHELL
        sudo cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
        sudo chmod 700 /home/vagrant/.ssh && chmod 600 /home/vagrant/.ssh/authorized_keys
        echo "Provisioning script completed on ansible-target-#{i}"
      SHELL
    end
  end
end

# -*- mode: ruby -*-
# vi: set ft=ruby :

# Read the local SSH public key
ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_ed25519.pub").first.strip

Vagrant.configure("2") do |config|
  # Base box
  config.vm.box = "debian/bullseye64"
  config.vm.box_check_update = false
  config.vbguest.auto_update = false

  # Nodes configuration
  nodes = {
    "master" => "10.0.0.10",
    "w1" => "10.0.0.11",
    "w2" => "10.0.0.12"
  }

  # Provision nodes
  nodes.each do |prefix, ip|
    config.vm.define prefix do |node|
      node.vm.hostname = prefix
      node.vm.network "private_network", ip: ip
      node.vm.network "forwarded_port", guest: 22, host: 2200 + ip.split('.')[3].to_i, id: 'ssh'
      node.vm.provider "virtualbox" do |vb|
        vb.name = prefix
        vb.memory = "2048"
        vb.cpus = "2"
      end

      # Basic setup
      node.vm.provision "shell", inline: <<-SHELL
        #{nodes.map { |name, ip_addr| "echo \"#{ip_addr} #{name} #{name}\" >> /etc/hosts" }.join("\n")}
        mkdir -p /home/vagrant/.ssh
        chown -R vagrant:vagrant /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
      # Setting up the locales otherwise ansible doesnt work fine from the get go 
        sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen
        locale-gen en_US.UTF-8
        update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
      SHELL

      # Install VirtualBox Guest Additions
      node.vm.provision "shell", inline: <<-SHELL
        apt update
        apt install -y linux-headers-$(uname -r) build-essential dkms
        VBOX_ISO="/vagrant/VBoxGuestAdditions_7.0.10.iso"
        mkdir -p /mnt/cdrom
        mount -o loop $VBOX_ISO /mnt/cdrom
        sh /mnt/cdrom/VBoxLinuxAdditions.run || echo "Failed to install Guest Additions. If everything else seems fine, you may ignore this."
        umount /mnt/cdrom
      SHELL

      # Install latest Python
      node.vm.provision "shell", inline: <<-SHELL
        apt update
        apt install -y python3 python3-pip
      SHELL

      # Additional provisioning for master node
      if prefix == "master"
        # Generate SSH key on master
        node.vm.provision "shell", inline: <<-SHELL
          if [ ! -f /home/vagrant/.ssh/id_ed25519 ]; then
            ssh-keygen -t ed25519 -f /home/vagrant/.ssh/id_ed25519 -N ''
            chown vagrant:vagrant /home/vagrant/.ssh/id_ed25519*
            # Copy the public key to the Vagrant directory
            cp /home/vagrant/.ssh/id_ed25519.pub /vagrant/master_id_ed25519.pub
          fi
          echo "#{ssh_pub_key}" >> /home/vagrant/.ssh/authorized_keys
        SHELL

        # Install latest Ansible on master
        node.vm.provision "shell", inline: <<-SHELL
          pip3 install ansible
          mkdir -p /etc/ansible
          echo "[nodes]" > /etc/ansible/hosts
          echo "localhost ansible_connection=local" >> /etc/ansible/hosts
          echo "#{nodes['w1']} ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/.ssh/id_ed25519" >> /etc/ansible/hosts
          echo "#{nodes['w2']} ansible_user=vagrant ansible_ssh_private_key_file=/home/vagrant/.ssh/id_ed25519" >> /etc/ansible/hosts
        SHELL
      else
        # Worker nodes
        node.vm.provision "file", source: "master_id_ed25519.pub", destination: "/home/vagrant/.ssh/master_id_ed25519.pub"
        node.vm.provision "shell", inline: <<-SHELL
          cat /home/vagrant/.ssh/master_id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
          echo "#{ssh_pub_key}" >> /home/vagrant/.ssh/authorized_keys
          chmod 600 /home/vagrant/.ssh/authorized_keys
        SHELL
      end
    end
  end
end


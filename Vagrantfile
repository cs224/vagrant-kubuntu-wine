# -*- mode: ruby -*-

# You can login to the master via:
# ssh-add ~/.vagrant.d/insecure_private_key
# ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no vagrant@192.168.56.101
# ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 2222 vagrant@localhost

# vagrant plugin install vagrant-reload
# vagrant plugin install vagrant-vbguest
## vagrant plugin install vagrant-disksize

# https://ostechnix.com/how-to-use-vagrant-with-libvirt-kvm-provider/
# vagrant plugin install vagrant-libvirt
# vagrant plugin install vagrant-mutate
# vagrant up --provider=libvirt
# export VAGRANT_DEFAULT_PROVIDER=libvirt
# virsh list
# vagrant destroy

# vagrant ssh -- -L 5432:localhost:5432


Vagrant.require_version ">= 2.3.4"

boxes = [
    {
        :name => "master",
        :eth1 => "192.168.56.101",
        :mem => "4096",
        :cpu => "2"
    },
]

bring_up_host_interface = <<SCRIPT
sudo ip link set dev enp0s8 up
SCRIPT

Vagrant.configure("2") do |config|

    # if Vagrant.has_plugin?("vagrant-vbguest")
    #   config.vbguest.auto_update = false
    # end

    # config.disksize.size = '15GB'

    config.ssh.forward_agent = true
    config.ssh.forward_x11 = true
    config.ssh.insert_key = false
    config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

    config.vm.synced_folder ".", "/vagrant"

    boxes.each do |opts|
        config.vm.define opts[:name] do |config|
            config.vm.hostname = opts[:name]

            config.vm.box = "bento/ubuntu-22.04"
            config.vm.box_version = "202309.08.0"
            config.vm.box_check_update = false

            config.vm.network :private_network, ip: opts[:eth1]
            config.vm.provision "shell", privileged: false, inline: bring_up_host_interface, keep_color: true, name: "bring_up_host_interface"

            config.vm.provider "virtualbox" do |virtualbox|
                virtualbox.gui = true
                virtualbox.customize ["modifyvm", :id, "--memory", opts[:mem]]
                virtualbox.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

                virtualbox.customize ['modifyvm', :id, '--clipboard-mode', 'bidirectional']
                virtualbox.customize ['modifyvm', :id, '--draganddrop', 'bidirectional']

                virtualbox.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga']
                virtualbox.customize ["modifyvm", :id, "--vram", "128"]
                virtualbox.customize ['modifyvm', :id, '--accelerate3d', 'on']
            end

            # Add `vagrant` to Administrator
            config.vm.provision :shell, inline: "sudo usermod -a -G sudo vagrant", keep_color: true, name: "Add `vagrant` to Administrator"

            # Define a closer download mirror
            config.vm.provision :shell, inline: "sudo cp /etc/apt/sources.list /etc/apt/sources.list.orig && sudo sed -i 's|http://us.|http://de.|g' /etc/apt/sources.list", keep_color: true, name: "Define a closer download mirror"

            # Update repositories
            config.vm.provision :shell, inline: "sudo apt update -y", keep_color: true, name: "apt update"

            # Upgrade installed packages
            config.vm.provision :shell, inline: "sudo apt upgrade -y", keep_color: true, name: "apt upgrade"

            # apt install linux-headers-$(uname -r) build-essential dkms
            config.vm.provision :shell, inline: "sudo apt install -y --no-install-recommends linux-headers-$(uname -r) build-essential dkms module-assistant", keep_color: true, name: "build-essential dkms module-assistant"
            # m-a prepare
            config.vm.provision :shell, inline: "sudo m-a prepare", keep_color: true, name: "m-a prepare"

            # Add desktop environment
            config.vm.provision :shell, inline: "sudo apt install -y plasma-desktop sddm language-pack-kde-de konsole", keep_color: true, name: "kubuntu-desktop"

            # Add Firefox
            #  https://www.ubuntuupdates.org/package/ubuntuzilla/all/main/base/firefox-mozilla-build
            config.vm.provision :shell, inline: "sudo apt install -y firefox", keep_color: true, name: "Add Firefox"

            # Install Docker
            config.vm.provision :shell, inline: "sudo curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh", keep_color: true, name: "Install Docker"
            config.vm.provision :shell, inline: "sudo usermod -aG docker vagrant", keep_color: true, name: "Add vagrant user to docker group"

            # Restart
            config.vm.provision :shell, inline: "sudo shutdown -r now", keep_color: true, name: "Restart"
        end
    end
end

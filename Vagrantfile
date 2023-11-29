# vagrant configuration

Vagrant.configure("2") do |config|

    # tierra config
    config.vm.define "tierra" do |tierra|
      tierra.vm.box = "debian/bullseye64"
      tierra.vm.hostname = "tierra.sistema.sol"
      tierra.vm.network :private_network, type: "static", ip: "192.168.57.103"
      tierra.vm.provider :virtualbox do |vbox|
          vbox.memory = 256
          vbox.linked_clone = true
      end
      tierra.vm.provision "shell", path: "provision.sh"
    end

    # venus config
    config.vm.define "venus" do |venus|
      venus.vm.box = "debian/bullseye64"
      venus.vm.hostname = "venus.sistema.sol"
      venus.vm.network :private_network, type: "static", ip: "192.168.57.102"
      venus.vm.provider :virtualbox do |vbox|
        vbox.memory = 256
        vbox.linked_clone = true
      end
      venus.vm.provision "shell", path: "provision.sh"
    end
end


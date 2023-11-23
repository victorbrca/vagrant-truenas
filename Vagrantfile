# https://app.vagrantup.com/ealvarenga/boxes/TrueNAS-SCALE
# Default credentials
#   Admin interface: root / vagrant
#   Console: root / vagrant
#   Shell: root / vagrant
#   Shell: vagrant / vagrant

VAGRANTFILE_API_VERSION = "2"

# Changes the default IP from 192.168.56.15
private_ip = "192.168.55.10"

##-- Global config -------------------------------------------------------------
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Disable the synced folder (no Guest Additions on image)
  config.vm.synced_folder '.', '/vagrant', disabled: true


  ##-- Machine -----------------------------------------------------------------
  config.vm.define "truenas" do |truenas|

    truenas.vm.box = "victorbrca/TrueNAS-13.0-U5.3"
    truenas.vm.network "private_network", ip: private_ip
    truenas.vm.provision :shell, :run => 'always', :inline => "ifconfig em1 inet #{private_ip} netmask 255.255.255.0"


    ##-- Provider --------------------------------------------------------------
    truenas.vm.provider "virtualbox" do |vb|

      # Use internal DNS from host
      vb.customize [ "modifyvm", :id, "--natdnshostresolver1", "on" ]
      vb.name = "TrueNAS13"
      vb.memory = "2048"

      # Add 3 x 2G disks
      (2..5).each do |i|
        file_to_disk = "./truenas-00#{i}.vdi"

        # Add disks
        unless File.exist?(file_to_disk)
          vb.customize ['createhd', '--filename', file_to_disk, '--size', 2 * 1024]
          vb.customize ['storageattach', :id, '--storagectl', 'AHCI', '--port', i, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
        end
      end
    end
  end
end
This is a Vagrant repo for a TrueNAS box that can be used for basic test/developing.

## Box Info

The box is configured with the information below.

**Vagrant Cloud:** [victorbrca/TrueNAS-13.0-U5.3 ](https://app.vagrantup.com/victorbrca/boxes/TrueNAS-13.0-U5.3)
**Vagrant Box Version:** 0.1.0
**TrueNAS Version:** TrueNAS-13.0-U5.3.

### Built-in credentials

+ **Admin interface**: root / vagrant
+ **Shell**: root / vagrant
+ **Shell**: vagrant / vagrant

### Sudo access

Vagrant needs to have passwordless sudo. Because TrueNAS saves everything in a SQL db, this is done via a 'init' script from the UI:

```
echo 'vagrant ALL=(ALL) NOPASSWD: ALL' >> /usr/local/etc/sudoers
```

### IP

You can change the IP by providing it in the Vagrantfile:

    truenas.vm.provision :shell,
                         :run => 'always',
                         :inline => "ifconfig em1 inet [private_ip] netmask 255.255.255.0"

> [!note]
> The example vagrant file included on this repo uses the variable `private_ip` for the IP

### Storage

Two drives are included in the image with the following layout:

```
vagrant@truenas:~ $ zpool status
  pool: boot-pool
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        boot-pool   ONLINE       0     0     0
          ada0p2    ONLINE       0     0     0

errors: No known data errors

  pool: data
 state: ONLINE
config:

        NAME                                          STATE     READ WRITE CKSUM
        data                                          ONLINE       0     0     0
          gptid/48061457-88a9-11ee-99a1-08002767d85d  ONLINE       0     0     0

errors: No known data errors
```

## Setup

You can easily start the box with:

```bash
vagrant init victorbrca/TrueNAS-13.0-U5.3
vagrant up
```

Or by manually adding to a Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "victorbrca/TrueNAS-13.0-U5.3"
end
```

### Example Vagrantfile

```ruby
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

    end
  end
end
```

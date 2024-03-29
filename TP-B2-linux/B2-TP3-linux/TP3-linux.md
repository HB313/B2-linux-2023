# TP3 Admin : Vagrant


## 1. Une première VM

### 🌞 Générer un Vagrantfile


```
hb313@fedora:~/work/vagrant$ mkdir test
hb313@fedora:~/work/vagrant$ cd test
hb313@fedora:~/work/vagrant/test$ vagrant init generic/debian10
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
hb313@fedora:~/work/vagrant/test$ ls
Vagrantfile
hb313@fedora:~/work/vagrant/test$ cat Vagrantfile
```

### 🌞 Modifier le Vagrantfile

```
hb313@fedora:~/work/vagrant/test$ nano Vagrantfile

```

```
Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2204"
  config.vm.box_check_update = false
  config.vm.synced_folder ".", "/vagrant", disabled: true

end
```

### 🌞 Faire joujou avec une VM

```
hb313@2a02-8440-6140-2cef-f2a9-4c33-3faa-cb97:~/work/vagrant/test$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'generic/debian10' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'generic/debian10'
    default: URL: https://vagrantcloud.com/api/v2/vagrant/generic/debian10
==> default: Adding box 'generic/debian10' (v4.3.10) for provider: virtualbox (amd64)
    default: Downloading: https://vagrantcloud.com/generic/boxes/debian10/versions/4.3.10/providers/virtualbox/amd64/vagrant.box
    default: Calculating and comparing box checksum...
==> default: Successfully added box 'generic/debian10' (v4.3.10) for 'virtualbox (amd64)'!
==> default: Importing base box 'generic/debian10'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: test_default_1705172061305_19165
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 5.2.0 r68940
    default: VirtualBox Version: 7.0
hb313@2a02-8440-6140-2cef-f2a9-4c33-3faa-cb97:~/work/vagrant/test$ vagrant ssh
sudo apt update
sudo apt install -y virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
vagrant@debian10:~$ vagrant status
-bash: vagrant: command not found
vagrant@debian10:~$ status
-bash: status: command not found
vagrant@debian10:~$ vagrant halt
-bash: vagrant: command not found
vagrant@debian10:~$ halt
-bash: halt: command not found
vagrant@debian10:~$ exit
logout
[sudo] password for hb313: 
Sorry, try again.
[sudo] password for hb313: 
sudo: apt: command not found
sudo: apt: command not found
hb313@2a02-8440-6140-2cef-f2a9-4c33-3faa-cb97:~/work/vagrant/test$ vagrant halt
==> default: Attempting graceful shutdown of VM...
hb313@2a02-8440-6140-2cef-f2a9-4c33-3faa-cb97:~/work/vagrant/test$ vagrant status
Current machine states:

default                   poweroff (virtualbox)

The VM is powered off. To restart the VM, simply run `vagrant up`
hb313@2a02-8440-6140-2cef-f2a9-4c33-3faa-cb97:~/work/vagrant/test$ 

```

## 2. Repackaging

### 🌞 Repackager la box que vous avez choisie

```
hb313@fedora:~/work/vagrant/test$ vagrant package --output super_box.box
==> default: Attempting graceful shutdown of VM...
==> default: Clearing any previously set forwarded ports...
==> default: Exporting VM...
==> default: Compressing package to: /home/hb313/work/vagrant/test/super_box.box
hb313@fedora:~/work/vagrant/test$ ls
super_box.box  Vagrantfile
hb313@fedora:~/work/vagrant/test$ vagrant box add super_box super_box.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'super_box' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/hb313/work/vagrant/test/super_box.box
==> box: Successfully added box 'super_box' (v0) for ''!
hb313@fedora:~/work/vagrant/test$ 


```

### 🌞 Ecrivez un Vagrantfile qui lance une VM à partir de votre Box

```
hb313@fedora:~/work/vagrant/test$ ls
super_box.box
hb313@fedora:~/work/vagrant/test$ vagrant init super_box.box
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
hb313@fedora:~/work/vagrant/test$ ls
super_box.box  Vagrantfile
hb313@fedora:~/work/vagrant/test$ nano Vagrantfile 
hb313@fedora:~/work/vagrant/test$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 6.1.48
    default: VirtualBox Version: 7.0
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
hb313@fedora:~/work/vagrant/test$ vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
hb313@fedora:~/work/vagrant/test$ vagrant ssh
Last login: Thu Jan 18 14:33:08 2024 from 10.0.2.2
[vagrant@rocky9 ~]$ exit
logout
hb313@fedora:~/work/vagrant/test$ 

```

## 3. Moult VMs

### 🌞 

voir fichiers Vagrantfile-3A et 3B
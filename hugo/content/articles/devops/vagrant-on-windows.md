---
title: "Vagrant on Windows"
date: 2020-11-24T11:29:58+01:00
authors: ["Jon Brookes"]
draft: false
---

Vagrant is a tool for orchestrating the creation of one or more virtual instances, typically based upon Linux distributions. This article is based upon Ubuntu 1804 but the Vagrantfiles used can be changed to use different distos or boxes - see https://app.vagrantup.com/boxes/search

Running Ubuntu 1804 or other linux distributions on Windows 10 can be achieved using 'Windows subsystem for Linux' ( WSL ) and this is often a good alternative to running a full blown VM using a hypervisor solution such as VirtualBox or Windows native HypverV. However, WSL is what it says on the tin - its a 'subsystem' and if your needing to test absolutely what happens on a Linux system nothing will be better than a booted running Linux system, albeit running within a virtualization platform. If your choosing to run Windows as a platform to develop Linux upon, Vagrant with Virtualbox or HyperV could be a useful tool to have at ones disposal.

It is assumed that all command line instructions within this article are run from within a Powershell command line running as an administrative user in either Powershell for desktop or Powershell Core.

## Prerequisites

A virtualisation platform, many people use Virtualbox :

https://www.virtualbox.org/wiki/Downloads

The downloaded installer will require administrative privilege and will require the system to be rebooted to work.

If you are a pro user of Windows 10 and use hyperv, virtualbox is not going to be an option as the 2 platforms cannot coexist on the same system. This is not a problem though as Vagrant supports hyperv amongst other providers - see https://www.vagrantup.com/docs/providers.

## Download and install

Downloading and installing Vagrant for windows is as simple as for other platforms from 

https://www.vagrantup.com/downloads

The downloaded installer will require administrative privilege and will also require a reboot to work. 

## Create a virtual instance of Ubuntu 1804

Create a directory to run a new Ubuntu 1804 linux configuration from and within this create a new file called `Vagrantfile` :

```
Vagrant.configure("2") do |config|
    config.vm.define "build01" do |build01|
        build01.vm.box = "generic/ubuntu1804"
        build01.vm..memory = "4096"
        build01.vm.hostname = "build01"
        build01.winrm.username = "vagrant"
        build01.winrm.password = "vagrant"
        build01.vm.network "public_network"
        build01.vm.provider "hyperv" do |h|
            h.cpus = 4
            h.maxmemory = 4096
        end
    end
end
```

start the image with 

```
vagrant up
```

if you are using hyperv, vagrant needs to be told to use this rather than the default and if the provider line has not been added to your Vagrantfile

```
vagrant up --provider=hyperv
```

if Vagrant is installed correctly together with virtualbox or hyperv this should complete and the machine will be available over ssh with the user 'vagrant' and password 'vagrant' but a quick way to connect to a console within the same directory is to run :

```
vagrant ssh
```

which gives us an ssh session in which we can access the linux system normally through the shell and `sudo -s` to switch to the root user etc ...

to exit, as normal we can just exit the shell with 'CNTRL-D'

## Managing vagrant boxes

vagrant has a whole bunch of options and to see these just run `vagrant` at the command line.

to finish using and delete the current running system, from within the current directory containing a Vagrantfile run

```
vagrant destroy
```

but it may also be convenient to come back to a running instance in its current state at a later time and for this we can 

```
vagrant suspend
```

to 'suspend' the current running image, which is at it suggests, suspends the running VM to be later resumed with :

```
vagrant resume
```

## Adding optional features

the following Vagrantfile adds some features to make working with an Ubuntu 1804 instance a bit more friendly :

```
Vagrant.configure("2") do |config|
    config.vm.define "docker01" do |docker01|
        docker01.vm.box = "generic/ubuntu1804"
        docker01.vm.provider "hyperv"
        docker01.vm..memory = "4096"
        docker01.vm.hostname = "docker01"
        docker01.winrm.username = "vagrant"
        docker01.winrm.password = "vagrant"
        docker01.vm.network "public_network", bridge: "Default Switch"
        docker01.vm.provider "hyperv" do |h|
            h.cpus = 4
            h.maxmemory = 4096
        config.env.enable
        config.vm.provision "shell", path: "script.sh"
        config.vm.synced_folder ".", "/vagrant", type: "smb", smb_password: ENV['SMBUserPassword'], smb_username: ENV['SMBUserName']
        end
    end
end
```

this uses a `script.sh` file, again in the same directory as is the Vagrantfile :

```
apt update -y
apt upgrade -y
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"    
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io   
usermod -aG docker vagrant
```

which is invoked by vagrant once the machine has been initially stood up - see the line `config.vm.provision "shell", path: "script.sh"` for the point at which this is run. The script goes on to update the newly created VM to have the latest updates available at the time and to install docker from the docker repositories.

Additionally, an 'SMB share' ( windows share ) is 'mounted' by the newly running VM - see the line `config.vm.synced_folder "." ...` where this is enabled. What this means is that files that are added / removed / edited within the ( linux shell ) under `/vagrant` are also available in the current vagrant directory that holds the Vagrantfile and vice versa, permitting the easy transference of files and data between the 2 running platforms.

On the system that this ran from, there is a default network set on the line `docker01.vm.network "public_network", bridge: "Default Switch"` which may need to be altered to suit other environments.

In the powershell session that we are running this from and to get the environment variables to be present and ready for this image to be build, first run something like :

```
$Env:SMBUserName = "<your user>@localhost"
$Env:SMBUserPassword = "<your password>"
```

and finally build the image with 

```
vagrant up
```







# References

https://www.vagrantup.com/docs/networking/public_network

https://stackoverflow.com/questions/44394725/how-do-i-set-the-smb-username-and-password

https://www.vagrantup.com/docs/synced-folders/smb.html#smb_username

https://www.nickhammond.com/configuring-vagrant-virtual-machines-with-env/

https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-7.1


# ml-setup

*Simple installation of machine learning and data science software*

The goal is to make the process of installing ML software on Linux dead easy to perform. Instead of maintaining some esoteric list of procedures that need to be peformed manually, this process is simple and fully automated. If the hardware is ready, you could install dozens of systems like this in a day - or thousands if you do it remotely.

Apps and libraries installed:

* Python 2.7
* Octave
* R, RStudio
* Nvidia graphics drivers
* CUDA libraries
* the cuDNN library
* Lua
* TensorFlow
* Theano
* Keras
* ... etc ...

The process is reproducible. The installer is idempotent - repeating the main install command at any later time will simply correct any deviations (uninstalled packages) and bring the system back to its desired state.

The process is based on standard configuration management techniques and tools (e.g. Ansible). Your DevOps folks are probably using something like this (on a much larger scale) to build and maintain your infrastructure.

## Prerequisites

### Hardware

A PC that's not very old.

* graphics card: a reasonably new Nvidia card
  * Compute capability >= 3.0 https://developer.nvidia.com/cuda-gpus
  * lots of video RAM (more is better)
* Make sure the system RAM >= video RAM (may not be always true if you have a card like the Titan X Pascal with 12 GB video RAM)

### Software

Ubuntu 14.04 Desktop amd64 version. For now you must use 14.04; some of the apps and packages have not been ported to v16 or newer.

Download the image: http://www.ubuntu.com/download/alternative-downloads

Just do a quick install with default settings. Don't get too fancy; simpler is better.

## Install the Machine Learning software

### Preparations

Make sure you have Internet access. Then fire up a terminal and become root:

```
sudo su -
```
Keep the root privileges for the whole duration of this process.

Enable the Ansible repository (the Ansible version in Ubuntu 14.04 is a little old, this will give you the latest stable):

```
apt-add-repository ppa:ansible/ansible
```

Refresh the apt cache:

```
apt-get update
```

Install Ansible, vim, and git:

```
apt-get install ansible vim git
```

Clone this repository from GitHub:

```
git clone https://github.com/FlorinAndrei/ml-setup.git
```

### Install ML software

Read the comments in `ml-setup/group_vars/all` and create/edit `~/.bash_profile` in your home directory accordingly. Logout, then log back in.

Switch to the repo folder:

```
cd ml-setup
```

Save your file and exit all apps - a reboot may happen during the process, so be prepared.

Run the installer:

```
ansible-playbook -i inventory install.yml
```

The installer will first apply all the OS updates and patches. If the kernel is updated, **IT WILL REBOOT YOUR MACHINE!!!** This step is not optional, so be prepared.

After the reboot, get back in the game:

```
sudo su -
cd ml-setup
ansible-playbook -i inventory install.yml
```

Now there are no updates to apply, so it will go quickly over that part.

At the end you should at least log out, then log back in - although to make sure everything is 100% operational you should reboot one last time. Linux doesn't require a lot of rebooting, but if you're pedantic or just very cautious it's worth doing this just once more (e.g. to make sure the driver has been installed properly).

Every time you run `ansible-playbook -i inventory install.yml` in the repo folder from now on, it will replay the process. If new packages are available, it will install the updated versions.

If you want to keep existing package versions (for stability reasons), you must edit `~/.bash_profile` and change `ansible_package_state` from `latest` to `present`. When you do that, subsequent runs will simply correct missing packages (uninstalled by mistake), but will not update existing packages. I recommend editing `ansible_package_state` to avoid surprises in the future. You can always update individual packages if you absolutely have to:

```
apt-get update
apt-get upgrade <package-name>
```

If you change `ansible_package_state` to `present`, you should also never run `apt-get upgrade` since it does the same thing as `latest` (apply all available upgrades). You should also disable the GUI update reminder on the desktop.

## Maintenance

Super-simple:

```
sudo su -
cd ml-setup
ansible-playbook -i inventory install.yml
```

That's it. It will replay the playbook and go through all steps, fixing issues if any.

Normally you should never do this again. Only replay the installer if you suspect there are issues with the ML packages - e.g. something got uninstalled by mistake. But the capability exists for those rare times when you do need it.

## Test it in a virtual machine

If you want to test the installer in a virtual machine before you commit it to bare metal, it's pretty easy. I've included a Vagrantfile for that purpose.

On any host (Mac, Linux or Windows), install [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/). Clone this repo. Then, from within the repo folder, run:

```
vagrant up
```

This will download an Ubuntu 14.04 image, create a virtual instance, launch it, and provision it with ansible and git. Then you can ssh into the virtual machine:

```
vagrant ssh
```
Within the instance, you'll find the repo files from the host exported live under `/vagrant`, and any updates within the instance are reflected instantly on the host. So become root and cd to the repo:

```
sudo su -
cd /vagrant
```

NOTE: If the repo directory from the host is not exported under /vagrant on the guest, check the VirtualBox guest additions package on the guest - if it's too old and out of sync with the guest additions on the host, the directory export may fail.

Now you can run `ansible-playbook -i inventory install.yml` as usual.

When you're done, just run `poweroff` as root and the instance will shut down. Or, to save space, after the instance has shut down and now you're back on the host, run:

```
vagrant destroy
```

This will delete the virtual instance and its associated virtual disk. However, the Ubuntu 14.04 image (called a "box" in Vagrantspeak) will continue to be stored within the Vagrant box files, so next time you run `vagrant up` it will be really quick to create a new instance from scratch. Don't hesitate to run `vagrant destroy` to start over from scratch with a clean instance if you make a mistake; recreating the new instance is a very quick process.

BOA Vagrant Installer
==

Using this script you can easily setup a local Barracuda/Octopus/Aegir development environment using Vagrant and Chef.

- https://drupal.org/project/boa
- https://drupal.org/project/barracuda
- https://drupal.org/project/octopus

If you setup the hostmaster remote import module (instructions at bottom) correctly, you can import sites from a remote octopus instance that shares the same octopus username as this local one.

View how this dev/local/staging workflow can work with this screencast https://vimeo.com/76546448

Requirements
-- 

- http://vagrantup.com/

Note: I recommend you install the vbguest plugin on the system to avoid problems with the file mounting:

    vagrant plugin install vagrant-vbguest 

Getting vagrant in place
--

Make sure you have added a ´base´ box already, see http://docs.vagrantup.com/v2/getting-started/

    vagrant box add base http://files.vagrantup.com/precise32.box

When you are ready, run the following (takes about 60 minutes)

    vagrant up

If you want to see the content in the browser on your system, you need to add the ip of the VM to ´/etc/hosts´, and let it point to the addresses, you are going to run from the vm.

Step by Step instructions
--

- 1.) Fork this repo https://github.com/iaminawe/boa-vagrant and then clone your copy locally

- 2.) (optional) Edit this file /cookbooks/barracuda/recipes/default.rb and replace reoctopus (my octopus username) with your octopus username (it's arbitrary but don't use underscores) and my cmsadmin@raisedeyebrow.com with your e-mail - you can read more about the other additional options here http://drupalcode.org/project/barracuda.git/blob/HEAD:/docs/INSTALL.txt NOTE: MAKE SURE YOU REPLACE ALL INSTANCES IN THE ENTIRE REPO IF YOU DO THIS (including file names).

- 3.) Run "Vagrant up" from within the folder and wait around 60 mins

- 4.) When its complete you should receive e-mails with links to your new aegir instances (check spam folder if you don't get anything)
- An alternative way to find the logins for barracuda is in /var/aegir/logs/install.log
- An alternative way to find the logins for an octopus instance is in /data/disk/octopus_user/logs/install.log

- 5.) In your /etc/hosts/ file point the domain names you would like to use (for main barracuda, octopus instance and each site) at 192.168.10.88
Its possible to setup a wilcard dns on mac osx but I have not succeeded in doing so yet.

    192.168.10.88 reoctopus.sub.aegir.local
    192.168.10.88 chive.aegir.local
    192.168.10.88 myamazingsite.aegir

- 6.) You can rename the domain to access an octopus instance by unlocking the hostname platform in aegir and then editing the site and adding additional site aliases.


Using Shared Folders
--
You can share folders transparently between guest and host by uncommenting this line in the Vagrantfile and following this pattern for other folders you may want to share.

    config.vm.synced_folder "~/workspace/platforms", "/data/disk/iaminaweoctopus/static", nfs: true

This host folder "~/workspace/platforms" will then be automatically mounted to "/data/disk/octopus_user/static" within the guest when the VM starts up.
It should not be enabled in the Vagrant file on the initial build as depending on Guest Additions versions, it can break the build.

The ~/workspace/platforms folder (shared folder) needs to exist before running the installer with shared folders.

These are the steps to get get a mounted shared folder working.

- 1.) Initial Vagrant Up builds the server
- 2.) Edit the Vagrantfile and uncomment the shared folder code.
- 3.) From the project folder - vagrant ssh in 
- 4.) Run "  sudo /etc/init.d/vboxadd setup", then type "exit" to logout the guest system
- 5.) Run vagrant reload to start vagrant and mount folder
- 6.) Type admin password when requested
- 7.) You can now make a test file in the VM static folder and check it appears in the host platforms folder

Troubleshooting
--
Guest additions versions can cause issues with shared folders working properly so if you get mount related errors when you run vagrant up
Run this in the guest VM

    sudo /etc/init.d/vboxadd setup

If the build timed out or chef failed for some reason. 

Try and run 

    vagrant provision
    
It will re-run the chef scripts and will likely repair a broken installation.

If the host machine crashes and when you start it up again the guest machine will not vagrant up and you get an error like
vagrant up failed, /dev/vboxnetctl: no such file or directory

    Follow these instructions http://stackoverflow.com/questions/18149546/vagrant-up-failed-dev-vboxnetctl-no-such-file-or-directory



Remote import
--

Remember to check whether ´/data/disk/octopus_user/.drush/provision/remote_import´ has been deleted.

If you want to be able to do remote imports, you need to do the following manually:

Copy the key to the remote server (also see http://larsolesen.dk/node/358)

    ssh-copy-id -i .ssh/id_rsa.pub octopus_user@remote-server

Go to ´admin/hosting/features´under ´Experimental´ and add remote import.
Go to ´Servers´ and add server. Choose hostmaster.

Optional Step: Symlink custom modules to your site
--

Additionally `~/workspace/modules and ~/workspace/themes` can be mounted into `/data/all/o_custom_modules` and `/data/all/o_custom_themes` by adding them to the vagrant file.

Follow the pattern of the default folder mount in the Vagrantfile to add these additional folders

If you want to use any of those on your platform, you can symlink those diretories into your `sites/all` folder.

    ln -s /data/all/o_custom_themes platforms/platformname/sites/all/themes/o_custom_themes
    ln -s /data/all/o_custom_modules platforms/platformname/sites/all/modules/o_custom_modules

Now you can use your usual development tools for your site.

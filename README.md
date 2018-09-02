# Preamble

This is an Ansible demo made to explain the basics of this formidable tool. 

This will deploy an AWS Lightsail instance and install all the requirements needed, then deploy a Wordpress on it. 

It goes from nothing and do all the job:

	Nothing -> create a lightsail instance -> create a DNS a record for the future wordpress

You will only need to manually install Python on it, in order to permit Ansible to access it.

Then, it will : 

	Install PHP/Apache/MySQL/LetsEncrypt/memcached
	Create the needed mysql db and the db's user
	Do all the config (Apache, MySQL, PHP, memcached)
	Generate SSL certificate
	Power up the all thing

This project is named "poc-ansible". You can rename it of course, but be sure you change all the occurences to not break the workflow. See "How to customize" part.

**Please note:** That is a simple demo of [Ansible](https://ansible.com). It can do so much more. Be sure to visit [Ansible documentation center](http://docs.ansible.com).

# Requirements

## AWS
You will need :

* An AWS account
* Python universal tool [awscli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
* An AWS Lightsail keypair
* Correct credentials in ~/.aws/credentials and ~/.aws/config

You can install awscli via pip:
	
	sudo pip install --upgrade awscli

You can generate a new Lightsail keypair via:

	aws lightsail create-key-pair --key-pair-name myKey

Copy it and put it in the repo, in the ssh folder. Finally, write the configuration in ssh/config (See How to customize part).

## Ansible

First, install Ansible :

On Mac:

	brew install ansible

On Linux:

	sudo apt install ansible

Or if you're a Redhat guy:

	sudo yum install ansible

Then, just go into the repo and do : 

	bin/requirements.sh 

This will install all the Ansible requirements that you'll need, in roles/ directory.

## Important

This project is thought for an ubuntu-like OS. I cannot guarantee that it will work on a Redhat, despite the fact that the roles used seems to work on Redhat.

# How to customize
To customize this project to feet your needs, you have many entrypoints. 

First of all, you have the ssh/config file, whitch is needed to allow the connection. You **have to customize it**, every ssh connection may be different, that's why the file is empty, and present in gitignore file.

	Host poc-ansible # the friendly name you choose for the server (poc-ansible for me)
	  HostName 4.5.34.2 # the server IP 
	  IdentityFile ssh/yourkey.pem # the key you choose to connect it
	  User ubuntu # the ssh user you want to use

Then, **only if you change the name "poc-ansible"**, modify the env/inventory.ini file like this:

	sed -i 's/poc-ansible/your-server-name/g' env/inventory.ini

And all the others files to feet your server name:

	sed -i 's/poc-ansible/your-server-name/g' play/*.yml tasks/*.yml env/group_vars/all.yml

**Do not forget** to rename also the general host var file:

	cd env/host_vars && mv poc-ansible.yml my-server-name.yml

This is very important to Ansible, so it can see and get your variables.

Finally, you can customize the following files:

	env/group_vars/all.yml
	env/host_vars/poc-ansible.yml

To choose your needed name, DNS choice, instance type, instance blueprint, IP address, Wordpress version etc.. For all tasks and plays.
 
# How to use

## Once you have customized it, you can go ! :) 

**(note: don't forget to do everything of the "Requirements" part above before this, especially running bin/requirements.sh)**

So, first thing first, we create the instance:

	ansible-playbook tasks/create-instance.yml

Once your instance is up, get its IP and put it in:

	ssh/config
	env/group_vars/all.yml

**Note:** you can get its ip via :

	aws lightsail get-instances

Then, we create the DNS. 

**The create-dns tasks supposes that your DNS in handled in AWS Route53. If not, you'll have to do it manually. Same thing for the delete-dns task.**

	ansible-playbook tasks/create-dns.yml

Your instance is now up and running, your dns is being propagated. 

Ansible is *agentless* but it only needs python to work on the remote server, to gather facts among others things. It's installed by the user_data shell script in the create-instance task. 

Your ansible setup is totally ready. 

Try it with:

	ansible all -m ping
	
	poc-ansible | SUCCESS => {
   		 "changed": false,
   	 	 "ping": "pong"
	}

Then, we update our ubuntu system (AWS only propose 16.04) with:

	ansible-playbook tasks/do-release-upgrade.yml

The server will reboot.

**Note:** Later, you have a task to update your system on a regular basis, just do:
	
	ansible-playbook tasks/update-sys.yml
	
## Good to go ! 

So, from here you can choose to launch every play manually or to run the all thing at once. That's on you, the all project is *idempotente*. If you want to run the all thing at once, just do:

	ansible-playbook main.yml
	ansible-playbook tasks/deploy-wp.yml

If not, go launch every plays one by one in the play directory:

	ansible-playbook play/hostname.yml
	(...)
	ansible-playbook play/ntp.yml
	(...)
	ansible-playbook play/mysql.yml
	(...)
	
Just be sure to launch it in the order which is defined in main.yml file. 

Finally, go to [https://the.dnsyou.choose/](), you should see the familiar wordpress installation landing page.

# Cleaning after your demo
If you didn't use it to deploy a production (whitch you can btw, just be sure to customize it as well), you can clean the all thing via :

	ansible-playbook tasks/clean/delete-dns.yml 
	ansible-playbook tasks/clean/delete-instance.yml
	
This will... Oh, I'm sure you get it ;-)

**As mentionned above,the delete-dns tasks supposes that your DNS in handled in AWS Route53. If not, you'll have to do it manually.**

# Troubleshooting
AWS Lightsail Ansible module do not allow to change much in your instance parameters. By default, you can only reach your Lightsail instance via HTTP and SSH.

So you have to add a new rule in the network instance, in the Lightsail console, to let HTTPS trafic reach your server. 

Don't forget it if you cannot access your HTTPS page ;-)

# Copyright and license

This software is copyright (c) 2018 by Sandro CAZZANIGA.

This is free software; you can redistribute it and/or modify it under the terms of the GNU GPLv3+ license. 

You'll find a copy of that license in this repo. 
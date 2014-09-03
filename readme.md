## Ansible Provisioning and Deployment
### Contents
1. What's included?
1. Initial Server Account/SSH Setup
2. Server Provisioning
3. Ongoing Deployment
4. SSH Gotchas / FAQ

# What's included?
Rbenv, Rails, Unicorn, NGINX, Redis, Node

Optional: Elasticsearch & Sidekiq

**Horizontal scaling** [load balancer, application server(s), db] setup only requires that you uncomment a few lines and make slight alterations to the ***ansible*** *hosts* file.

# 1. Initial server setup

After adding new server(s) to the ***ansible*** *hosts* file in the ansible root directory you will need to add your git repo, app name, domain name and db password to the group_vars/vms file. This step ensures that all of our SSH & User Account info is set.

![Edit the group_vars/vms config file](https://www.evernote.com/shard/s293/sh/2421f85b-8027-4520-af10-e290e7b9d581/56430a58ad379b1f10860165e0b0c8b8/res/c7cee372-23a3-41ef-be7a-0d995abe7ce7/skitch.png?resizeSmall&width=832)

Run:

`$ ansible-playbook -l SERVER_NAME config.yml --tags "common" -e "ansible_ssh_user=USER" --ask-pass`
* Ommit -l if you are setting up multiple servers at one time.
* Ommit --ask-pass if your public key has already been setup with remote servers

# 2. Server provisioning

Then run:

  `$ ansible-playbook config.yml`
  
#### Optional:  
If you want sidekiq or elastisearch  uncomment their references from the following files:
- config.yml
- deploy.yml
- roles/app_install/main.yml
- roles/app_deploy/main.yml

If you want a multiple server setup (load balancer, multiple app servers, db server) 

1. Uncomment lines refering to "nginx\_load\_balancer" and "pg\_ssh\_tunnel" from:
  - config.yml
  - deploy.yml
2. Make sure the *hosts* file has a single unique server under [loadbalancers] and [dbservers]



# 3. Ongoing deployments
After every git push run:

  `$ ansible-playbook deploy.yml`

- --tags "reboot" restarts all services including elasticsearch
- by default deployment pulls git master (if changed), runs migrations (if changed), then restarts app server (+ sidekiq if applicable) and reloads nginx conf files.


# FAQ / SSH issues / TODO
## FAQ
- How do I install Ansible?
  - `brew install ansible`
- List of common command line flags
  - Limit which servers playbook is run on `ansible-playbook -l SERVER_NAME,SERVER_NAME2|GROUPNAME config.yml`
  - Limit which rules are run `ansible-playbook config.yml --tags "nginx"`
  - Use text password instead of public key`ansible-playbook config.yml --ask-pass`
- How do I run multiple applications on a single server?
  - Just reclone the ansible directory for you new app and use the default single server config with your app specific group_vars/vms values and then deploy independently, no config should overwrite.
- How do I run an adhoc command? 

  >`$ ansible webservers -m shell -a 'echo $TERM'` This runs "echo $TERM" on all webservers.

  > Now to run the command on all servers in a group, in this case, webservers, in 10 parallel forks:
  > `$ ansible webservers -a "/sbin/reboot" -f 10`
  > /usr/bin/ansible will default to running from your user account. 
  
  > If you do not like this behavior, pass in “-u username”. If you want to run commands as a different user, it looks like this:
  > `$ ansible webservers -a "/usr/bin/foo" -u username`
  
  > Often you’ll not want to just do things from your user account. If you want to run commands through sudo:
  > `$ ansible webservers -a "/usr/bin/foo" -u username --sudo [--ask-sudo-pass]`
  > Use --ask-sudo-pass (-K) if you are not using passwordless sudo. This will interactively prompt you for the password to use. Use of passwordless sudo makes things easier to automate, but it’s not required.

  > It is also possible to sudo to a user other than root using --sudo-user (-U):
  > `$ ansible webservers -a "/usr/bin/foo" -u username -U otheruser [--ask-sudo-pass]`

<a name="sshissues"></a>

## SSH issues
To avoid the constant management of public keys between server, the playbooks take advantage of SSH: *ForwardAgent*, you may need to add the below file to your dev box to make these recipes work (only use this for testing, ForwardAgent * is dangerous).

**~/.ssh/config**  

  Host 127.0.0.1
    ForwardAgent yes

  Host *
    ForwardAgent yes
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null


## TODO
- db backups
- db upload from local file
- cross server PEM sharing
  - don't rely on ForwardAgent ssh tunnel for service fowarding (until this change is made, PGres, etc. won't work on server reboot until you have run deploy.yml)
- ssh tunnel role for elasticsearch server support
- add ssl support w/ nginx
- app teardown (right now you just need to run a remote command to delete folders/disable services)

# Bonus round!
1. Install Vagrant & Virtualbox
2. Run `vagrant up` from ansible root directory
3. Ask me for git repo access to the "Skelly_Rails" project
4. Make sure your `~/.ssh/config` file matches the [SSH Issues](#sshissues) fix.
5. Add to your local host:
  `192.168.33.10 dev.com`
  `192.168.33.11 dev2.com`
  `192.168.33.12 dev3.com`
6. Run the commands in "Initial server setup" and "Server Provisioning" to launch.
7. Enjoy the preconfigured sandbox!
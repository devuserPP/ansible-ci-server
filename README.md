# CI environment Ansible scripts

## Execute playbooks

The main playbook is `site.yml` which imports all the others and it can be executed this way: 


```
ansible-playbook -i inventory -b site.yml -u ansible -v -k
```

Of course you can run just separate playbooks like `sonarqube.yml` but keep in mind that the playbooks should be executed in the order as defined in `site.yml` because eg. Sonarqube won't run without Docker installed on the machine and nginx will crash if all the referenced services aren't running.


## Ansible scripts structure

* __/site.yml__ - the main playbook
* __/*.yml__ - other playbooks referenced from site.yml
* __/inventory__ - an inventory files defining remote servers
* __/group_vars/all.yml__ - defines variables used in playbooks for all nodes
* __/roles/__ - roles are kind of subscripts referenced from playbooks
* __/roles/*/defaults/main.yml__ - variables used by a role are defined here
* __/roles/*/tasks/main.yml__ - the tasks the role executes


## Tips & Tricks

#### HTTP proxy
Our CI server isn't connected to the Internet directly but uses an HTTP proxy so if an Ansible task or Docker container needs an access to the Internet you must set environment variables `http_proxy` and `https_proxy` for it.


## CI/CD server

### Services structure

We use nginx as a reverse proxy for all requests to our systems.

```
---> nginx /
       | ---> Sonarqube /sonarqube
       | ---> Kibana /kibana
       | ---> pgAdmin4 /pgadmin
       | ---> Jenkins /jenkins
```

### Docker

All the services are running inside Docker containers and are connected to a Docker network `network.exmaple.com`. Use fully qualified names when referencing services inside the network eg. `sonarqube.network.exmaple.com` for the Sonarqube service.

Persistent data are kept in `/data/<service name>/` directory and mounted to the Docker containers as volumes.


## Make a node to be managable by Ansible

Ansible uses SSH and a user with sudo rights for connecting to managed nodes.  
We use a user called `ansible` in our environment so before a node can be managed by Ansible it must contain the user. 


```
# add a new user just for Ansible
adduser ansible

# add the user to wheel group so they can use sudo
usermod -a -G wheel ansible

# allow the ansible user to run sudo without prompt for password
echo "ansible         ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers.d/ansible
```

Ansible also requires the node to contain Python > 2.6 but it should be installed in RHEL we use by default.


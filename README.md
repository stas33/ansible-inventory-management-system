# This is the deployment guide of the inventory management application using Ansible 

# Ansible deployment
## Clone projects locally
```bash
git clone https://github.com/stas33/inventory-management-project.git
git clone https://github.com/stas33/ansible-inventory-management-system.git
```
## Edit inventoryproject/.env file to define
```vim
SECRET_KEY='test123'
DATABASE_URL='postgres://dbuser:pass123@db:5432/test_db'
```

## Setup of vms, ssh keys, installation
- Ssh from local pc to the first virtual machine
- Create ssh connection by generating a new ssh key
- Install ansible in the first virtual machine (https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
## To run and configure the ansible project locally
Procedure
* create an inventory file (e.g. hosts or hosts.yaml) that holds the remote hosts that ansible will handle.
* run 
```bash
ansible -m ping all
```
to check connectivity
* run testing environment
```bash
cd vagrant
vagrunt up
vagrant ssh-config >> ~/.ssh/config
```
* run a playbook
```bash
ansible-playbook -l database playbooks/database.yml
```

## Vault
* create a file that holds the **secret**
```bash
touch playbooks/vars/api_key.yml
```
* encrypt the file
```bash
ansible-vault encrypt playbooks/vars/api_key.yml
```
* run task that needs this file
```bash
ansible-playbook playbooks/use-api-key.yaml --ask-vault-pass
```
and you will be asked to provide the password
* edit the encrypoted file with
```bash
ansible-vault edit playbooks/vars/api_key.ym
```
* use stored password to decrypt
create a file that holds the password with 600 permissions
```bash
vim ~/.ansible/vault_pass.txt
chmod 600 ~/.ansible/vault_pass.txt
```
```bash
ansible-playbook playbooks/use-api-key.yaml --vault-password-file  ~/.ansible/vault_pass.txt
```
* postgres
install postgresql role
```bash
ansible-galaxy install geerlingguy.postgresql
```
## Docker
```bash
ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.pip
```
## Jenkins
```bash
ansible-galaxy install geerlingguy.jenkins
ansible-galaxy install geerlingguy.java
```

## Setup of credentials, rules, jenkins
- Create a second virtual machine and add the ssh key
- Add inbound port rules for ports 8000, 5432, 443, 80 in the second virtual machine
- Ssh to the second virtual machine
- Install ansible in the second virtual machine
- Login to jenkins server (in first virtual machine)
- Go to Dashboard->Manage Jenkins->Manage Credentials->Add
- Select Ssh username with private key->Enter id, username and paste private key
## Clone ansible project on Jenkins
- Go to Dashboard->Freestyle project
- Select Git->Repository url(https://github.com/stas33/ansible-inventory-management-system.git)->branch master
- Select Github hook trigger for GITScm polling
- Click Save and build
## Setup of pipeline
- Go to Dashboard->New Item->Pipeline project
- Select Github hook trigger for GITScm polling
- Select Definition->Pipeline script from SCM
- Select Scm->Git
- Paste repo url (https://github.com/stas33/inventory-management-project.git)
- Set branch to main
- Set script path to Jenkinsfile
- Click save and build
- Run http://<put second virtual machine's ip here>:8000/
- Go to the project's repo in the second virtual machine and run  python manage.py create superuser

## Links
* [apt module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
* [file module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
* [copy module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
* [service module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)
* [debconf module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debconf_module.html)

* [ansible postgres role](https://galaxy.ansible.com/geerlingguy/postgresql)
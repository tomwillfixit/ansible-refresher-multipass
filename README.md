# ansible-refresher-multipass

Using Multipass to create environments to test ansible playbooks against.

# Install Multipass

Instructions can be found [here](https://multipass.run/).

# Install Ansible
```
MacOS : brew install ansible

Ubuntu : apt install ansible
```

# Create 2 environments

The following commands will start 2 VMs based on Ubuntu 20.04. Each VM will be assigned 1 Gig of Memory, 1 cpu and 5 Gig of disk.

```
multipass launch --mem 1G --cpus 1 --disk 5G --name ansible1 20.04
multipass launch --mem 1G --cpus 1 --disk 5G --name ansible2 20.04
```

To get the IP Address of the VMs run : 

```
multipass ls

```

# Create Hosts file

Ansible uses this file by default to find the host IP and details required for the SSH connection.

Open /etc/ansible/hosts using your favourite text editor and add : 

```
[dev]
192.168.64.11
192.168.64.12

[dev:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

# Setup SSH keys on both environments

Log into each environment and add the contents of your ~/.ssh/id_rsa.pub from the host to the authorized_keys file.

```
multipass shell <environment_name>
sudo bash
vim ~/.ssh/authorized_keys

Append in the contents of the ~/.ssh/id_rsa.pub file from your host.

```

# Setup SSH options in Ansible

Before we run our first playbook there is one last step to perform. Ansible relies on SSH to connect to each environment and run commands. SSH has many options and these can be defined in the /etc/ansible/ansible.cfg file.

Open /etc/ansible/ansible.cfg using your favourite text editor and add : 

```
[defaults]
host_key_checking = False

[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no

```

The ssh_args field tells Ansible which SSH options to use when connecting to an environment. 

# Run a simple playbook

Create a playbook called : my_first_playbook.yml and append in the following :
```
---
- hosts: all
  tasks:
    - name: install apache2
      apt: name=apache2 update_cache=yes state=latest
```

To execute this playbook against both environments run :

sudo ansible-playbook -u root my_first_playbook.yml 

Example Output :
```
PLAY [all] ******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [192.168.64.11]
ok: [192.168.64.12]

TASK [install apache2] ******************************************************************************************************************************************************************
ok: [192.168.64.11]
ok: [192.168.64.12]

PLAY RECAP ******************************************************************************************************************************************************************************
192.168.64.11              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.64.12              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

At this point you have installed apache on your 2 environments.  You can re-run the command and ansible will detect that apache is already installed and skip the install step.

# Note

After playing about with Multipass for a bit I hit an issue with being unable to start VMs based off image 20.04. To resolve this I had to remove the downloaded image and re-download it.

```
sudo ls /var/root/Library/Caches/multipassd/vault/images/focal-20200804
ubuntu-20.04-server-cloudimg-amd64-initrd-generic	ubuntu-20.04-server-cloudimg-amd64-vmlinuz-generic	ubuntu-20.04-server-cloudimg-amd64.img

bash-3.2# rm -rf /var/root/Library/Caches/multipassd/vault/images/focal-20200804
```



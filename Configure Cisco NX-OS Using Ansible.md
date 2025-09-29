
# Prepare the switches

- Assign an IP address to mgmt interface, create an admin account, enable ssh on the switches

```
interface mgmt0
  vrf member management
  ip address 192.168.43.28/24

username admin password admin role network-admin
feature ssh
```

Test ssh access from the terminal before installing ansible

ssh admin@192.168.43.28

# Install ansible

```bash
# install pip for Python 3
sudo apt install python3-pip

# If ansible will be used inside virtual environment install venv
pip3 install virtualenv

# create and activate the virtual environment ansible and
python3 -m virtualenv ansible
source ansible/bin/activate

# install ansible
python3 -m pip install ansible

# In order to use the paramiko connection plugin or modules that require paramiko, install the required module

python3 -m pip install paramiko
python3 -m pip install ansible-pylibssh

# test if ansible is running ok
ansible localhost -m ping
```

# Create the directory structure

```bash
# create a new directory
mkdir ansible-lab
cd ansible-lab
```


```bash
# create ansible.cfg. This is going to override default settings
nano ansible.cfg
[defaults]
inventory= hosts.yml
host_key_checking = false
retry_files_enabled = false
interpreter_python =/usr/bin/python3
```

```bash
# create inventory file. This lab will use cli connection type
nano hosts.yml

[switches]
sw1 ansible_host=192.168.43.28
sw2 ansible_host=192.168.43.29

[all:vars]
ansible_user= "admin"
ansible_password= "admin"
ansible_connection= network_cli
ansible_network_os= cisco.nxos.nxos
```

# Playbook - Get details from switch Nexus

```bash
# this example will display some details like hostname and nxos version
nano get_swnx_details.yml

---
- name: "Get details from switch Nexus"
  hosts: all
  tasks:

    - name: "Display all hosts variables"
      debug:
        var: ansible_facts
```

# Playbook - Backup the config

```bash
---
- name: "Backup current switch nexus config"
  hosts: all
  tasks:
  
    - name: "Backing up the configuration"
      cisco.nxos.nxos_config:
        backup: yes
      register: backup_nxos_location
      when: ansible_network_os == 'cisco.nxos.nxos'
```

# Create vlans

```bash
---
- name: "Configure vlan"
  hosts: all
  gather_facts: no
  tasks:
  
    - name: "Create vlans"
      cisco.nxos.nxos_vlans:
        config:
          - vlan_id: 10
            name: VLAN10
          - vlan_id: 20
            name: VLAN20
        state: merged
```


# ansible_test

Set up SSH Keys:

1. Copy personal SSH Key to the server<br>
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP><br>
2. Create ansible SSH Key<br>
    ssh-keygen -t ed25519 -C "ansible" || (-t stands for type, -C stands for comment)<br>
    Note: rename the key to "ansible"<br>
3. Copy ansible SSH Key to the server<br>
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP><br>
4. Making sure OpenSSH uses the right key:<br>
    ssh -i ~/.ssh/ansible <SERVER-UP><br>

Set up Git:

1. Create a Respository
2. git clone <Git-Link>
3. Edit files
4. git add <Edited File>
5. git commit -m "commit-comment, what you have done"

Starting with Ansible:

1. First install ansible however you like
2. Create your first inventory:<br>
    This file contains a list of your managed nodes/hosts, either their DNS or their IP

3. First Ansible command:<br>
    ansible all --key-file ~/.ssh/ansible -i inventory -m ping<br>
        "--key-file"" is the ssh key that is used<br>
        "inventory" is the file with the IP addresses<br>
        "-m ping" is the module that is used
4. Creating an ansible.cfg file to shorten the command:<br>
    [defaults]<br>
    inventory = inventory<br>
    private_key_file = ~/.ssh/ansible
5. Now you can leave out keyfile and inventory:<br>
    ansible all -m ping
6. Another command to see all managed nodes/hosts:<br>
    ansible all --list-hosts
7. Another command, this time a bit more useful:<br>
    ansible all -m gather_facts<br>
        you basically see EVERYTHING of the servers<br>
        --limit <IP-ADDRESS> limits it to the one server

Running elevated ad-hoc Commands

1. Tell ansible to use sudo
    ansible all -m apt -a update_cache=true --become --ask-become-pass<br>
        -m apt : allows to work with apt-packages on a debian system<br>
        -a update_cache=true : essential makes sudo apt update to the managed node/hosts<br>
        --become : elevate the privileges of ansible (default is sudo)<br>
        --ask-become-pass : needed to add a password<br>
            Docs for the apt-module: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
2. Installing a package via ansible:
    ansible all -m apt -a name=vim-nox --become --ask-become-pass
        -m apt : allows to work with apt-packages on a debian system<br>
        -a name=vim-nox : the name of the package, here its vim - a text editor<br>
            doing the same with "tmux", another package<br>
        --become : elevate the privileges of ansible (default is sudo)<br>
        --ask-become-pass : needed to add a password
3. Making sure the package is the latest version possible:<br>
    ansible all -m apt -a name="snapd state=latest" --become --ask-become-pass<br>
        snapd : is another package to install<br>
        state=latest : is used to update that package
4. Installung all upgrades on the managed Node/Host:<br>
    ansible all -m apt -a "upgrade=dist" --become --ask-become-pass<br>
        -a upgrade=dist : is an argument that upgrades all packages on the managed Note/Host

Writing the first playbook:

1. Create a .yml file:
    kate install_apache.yml | Here you can just name your playbook however you like, ofcourse its best to give it a descriptive name
2. The first Playbook is written like this:
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2

    It is important to alight the lines to eachother(hosts, become, tasks and then name & apt). the - shows the start of a new block
    the first "name" is simply the name of the tasks that will be displayed
    the second "name" is the name of the package that is going to installed with "apt"
        You can add more tasks by following the same pattern:
        - name: Descriptive name of the task
          apt:
            name: name-of-the-package
            state: latest (to ensure we always install the latest package, can ofcourse be changed if u want a specific version) | absent can be used to remove the packages
3. Using the first playbook:
    ansible-playbook --ask-become-pass install_apache.yml
        "install_apache.yml" can be any .yml file that u have created



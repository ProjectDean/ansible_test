## ansible_test

##### Note: We are using debian and apt, if you dont use these on your servers, watch: [Video Link](https://www.youtube.com/watch?v=BF7vIk9no14&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&index=7)
---

### Set up SSH Keys:

1. Copy personal SSH Key to the server<br>
    `ssh-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>`
2. Create ansible SSH Key<br>
    `ssh-keygen -t ed25519 -C "ansible"` || (-t stands for type, -C stands for comment)<br>
    Note: rename the key to "ansible"
3. Copy ansible SSH Key to the server<br>
    `ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>`
4. Making sure OpenSSH uses the right key:<br>
    `ssh -i ~/.ssh/ansible <SERVER-IP><br>`

### Set up Git:

1. Create a Respository on github
2. Clone your Respository to your local machine:<br>
    `git clone <Git-Link>`
2. Edit files / Work with the Files
3. Add your edited files via git-command:<br>
    `git add <Edited File>` || (`git add .` adds all files in the directory you are in right now)
4. Commit your changes:<br>
    `git commit -m "comment, what you have done"`
5. Push your changes to the Respository:<br>
    `git push origin master`

### Starting with Ansible:

1. First install ansible however you like
2. Create your first inventory:<br>
    This file contains a list of your managed nodes/hosts, either their DNS or their IP and is a simple file called "inventory"<br>
    This is usually in a list of all the < ip-addresses > of the hosts/nodes you want to manage.<br>

3. First Ansible command:<br>
    `ansible all --key-file ~/.ssh/ansible -i inventory -m ping`<br>
        `--key-file` : is the ssh key that is used<br>
        `inventory` : is the file with the ip-addresses<br>
        `-m ping` : is the module that is used
4. Create a ansible.cfg file to shorten the command and fill it with:<br>
    [defaults]<br>
    inventory = inventory<br>
    private_key_file = ~/.ssh/ansible
5. Now you can leave out `--key-file` and `inventory`:<br>
    `ansible all -m ping`
6. Another command to see all managed nodes/hosts:<br>
    `ansible all --list-hosts`
7. Another command, this time a bit more useful:<br>
    `ansible all -m gather_facts`<br>
        you basically see EVERYTHING of the servers<br>
        `--limit <IP-ADDRESS>` limits it to the one server

### Running elevated ad-hoc Commands

1. Tell ansible to use sudo
    `ansible all -m apt -a update_cache=true --become --ask-become-pass`<br>
        `-m apt` : allows to work with apt-packages on a debian system<br>
        `-a update_cache=true` : essential makes sudo apt update to the managed node/hosts<br>
        `--become : elevate` the privileges of ansible (default is sudo)<br>
        `--ask-become-pass` : needed to add a password<br>
    > Docs for the apt-module: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
2. Installing a package via ansible:
    `ansible all -m apt -a name=vim-nox --become --ask-become-pass`<br>
        `-m apt` : allows to work with apt-packages on a debian system<br>
        `-a name=vim-nox` : the name of the package, here its vim - a text editor<br>
        `--become` : elevate the privileges of ansible (default is sudo)<br>
        `--ask-become-pass` : needed to add a password
3. Making sure the package is the latest version possible:<br>
    `ansible all -m apt -a name="snapd state=latest" --become --ask-become-pass`<br>
        `snapd `: is another package to install<br>
        `state=latest` : is used to update that package
4. Installing all upgrades on the managed Node/Host:<br>
    `ansible all -m apt -a "upgrade=dist" --become --ask-become-pass`<br>
        `-a upgrade=dist` : is an argument that upgrades all packages on the managed Note/Host

### Writing the first playbook:

1. Create a .yml file:<br>
    `kate install_apache.yml`<br>
    Here you can just name your playbook however you like, ofcourse its best to give it a descriptive name<br>
    Note: Kate is the text-editing program you use and by typing that command, you create or open the file you wrote
2. The first Playbook is written like this:
<code>
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
</code>

    It is important to align the lines to eachother(hosts, become, tasks and then name & apt). the - shows the start of a new block<br>
    the first `name` is simply the name of the tasks that will be displayed<br>
    the second `name` is the name of the package that is going to be installed with `apt`<br>
    You can add more tasks by following the same pattern:<br>

<code>
  - name: Descriptive name of the task
    apt:
      name: name-of-the-package
      state: latest
    when: ansible_distribution == "Debian"
</code>
    `state` : is used to determite what package is used or what we do with it. `latest` is used to install the latest package, `absent` is used to remove the package<br>
    `when` : is used as a condition, only if Debian is the OS of the Host, the task will be done(in this case, the package will be installed) <br>
    `apt` : dont forget to change this to the package-management of your OS<br>

3. Using the first playbook:<br>
    `ansible-playbook --ask-become-pass install_apache.yml`<br>
    `install_apache.yml` can be any .yml file that u have created<br>

### Improving the playbook:

1. Instead of writing a new block for every package we are installing, we can combine them into one task:
    - name: install apache2 package and php packages
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Debian"
2. you can use variables for your package-names:
    "{{ variable_name }}"
    You also have to add the variable to your inventory:
    < IP-ADDRESS > variable_name1=your_os_packagename1 variable_name2=your_os_packagename2
    Example:
    95.217.185.39 apache_package=apache2 php_package=libapache2-mod-php
3. Use package instead of apt to make your task universal - ansible will use whichever packagemanager is installed on the managed Node/Host

### Targeting Specific Nodes:
1. You can create Groups:
    [group_name]
    < IP-ADDRESS0 >
    < IP-ADDRESS1 >
2. You can target these groups in your playbook by changing:
    - hosts: all
    to:
    - hosts: group_name
    Example in "site.yml"
3. You can change "tasks:" to "pre_tasks:" to make sure the "pre_tasks" are run first, bevore other plays

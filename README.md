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
    `ssh -i ~/.ssh/ansible <SERVER-IP>`

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

```
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
```

It is important to align the lines to eachother(hosts, become, tasks and then name & apt). the - shows the start of a new block<br>
the first `name` is simply the name of the tasks that will be displayed<br>
the second `name` is the name of the package that is going to be installed with `apt`<br>
You can add more tasks by following the same pattern:<br>

```
  - name: Descriptive name of the task
    apt:ansible-playbook --ask-become-pass
      name: name-of-the-package
      state: latest
    when: ansible_distribution == "Debian"
```

`state` : is used to determite what package is used or what we do with it. `latest` is used to install the latest package, `absent` is used to remove the package<br>
`when` : is used as a condition, only if Debian is the OS of the Host, the task will be done(in this case, the package will be installed) <br>
`apt` : dont forget to change this to the package-management of your OS<br>

3. Using the first playbook:<br>
    `ansible-playbook --ask-become-pass install_apache.yml`<br>
    `install_apache.yml` can be any .yml file that u have created<br>

### Improving the playbook:

1. Instead of writing a new block for every package we are installing, we can combine them into one task:

```
    - name: install apache2 package and php packages
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Debian"
```
2. you can use variables for your package-names:
    `"{{ variable_name }}"`
    You also have to add the variable to your inventory:
    `< IP-ADDRESS > variable_name1=your_os_packagename1 variable_name2=your_os_packagename2`
    Example:
    `95.217.185.39 apache_package=apache2 php_package=libapache2-mod-php`
3. Use `package` instead of `apt` to make your task universal - ansible will use whichever packagemanager is installed on the managed Node/Host
4. You can add `tags:` to add metadata to the different play/task-blocks, with these tags you can target the specific Block<br>
    Example:<br>
```
 - name: install apache2 package and php packages
   tags: apache,web,php
    apt:
      name:
        - apache2
        - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Debian"
```
With: `ansible-playbook --list-tags site.yml` you cann look up all the tags that are used in the sitem.yml-file.<br>
With `ansible-playbook --tags apache --ask-become-pass site.yml` only the Taskblocks with the tag `apache` would run.<br>
Note: With the tag `always`, the task will always be run.<br>
### Targeting Specific Nodes:
1. You can create Groups:
```
    [group_name]
    < IP-ADDRESS0 >
    < IP-ADDRESS1 >
```
2. You can target these groups in your playbook by changing:
    `- hosts: all`
    to:
    `- hosts: group_name`
    Example in "site.yml"
3. You can change `tasks:` to `pre_tasks:` to make sure the "pre_tasks" are run first, before other plays/tasks
### Managing Files
1. Create another Task, that is used to copy a file to your Node/Host:
```
  - name: copy default html file for site
    tags: apache
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644
```
`copy` is the module name to copy a file to the Node/Host<br>
`src` is the source of the file you want to copy on to the Node<br>
`dest` is the destination of the copied file<br>
`owner` and `group` sets the owner and the group of the file to the user "root"<br>
`mode` sets the permission the file has once its on the Node/Host<br>

2. Create another Task, to download and then copy a file to your Node/Host:
```
- hosts: all #if you have more than one or want to target a specific one, just name another group u set in the inventory file
  become: true
  tasks:

  - name: install unzip
    package:
      name: unzip
  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      mode: 0755
      owner: root
      group: root
```
`unzip` is used to allow the Node/Host to unzip files/archives from the commandline<br>
`unarchive` is the task that is used in ansible to unzip the file you are downloading and to install it<br>
`src` is where the file comes from, this time from a downloadlink<br>
`remote_src` tells ansible that it is indead a remote source file that needs to be downloaded<br>

### Managing Services
1. Create another Task, this time to start the httpd Service
```
  - name: start httpd service (on something like CentOS)
    tags: apache,centos, httpd
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "CentOS"

  - name: restard httpd (CentOS)
    tags: apache,centos,httpd
    service:
      name: httpd
      state: restarted
    when: httpd.changed
```
We are now using `service` instead of `apt` and in `state` we now have `started` - This starts the service we have named.<br>
`state=restarted` would restart the service.<br>
`enabled: yes` is used so that the service is getting started, even after the host/node is restarted.<br>
`register: httpd` can be used to save the task into a variable.<br>
`when: httpd.changed` can be used if you want to ristrict the task to only run when the variabled-task changed anything.<br>
`changed_when: false` can be added so the change in a task wont get registered as a change(nice for updates, since that happens so often).<br>

### Managing User
1. Create another Task to create a new user on `all` hosts/nodes:
<details>
    <summary>Code</summary>

    ```
    - hosts: all
    become: true
    tasks:

    - name: create Siko user
        tags: always
        user:
        name: siko
        groups: root
    ```

</details>
2. Now adding the ssh key for the new user
<details>
    <summary>Code</summary>

    ```
    - name: add ssh key for siko
    tags: always
    authorized_key:
      user: siko
      key: "< PUBLIC SSH-KEY >"
    ```

</details>
3. Giving the new user the permissions to use sudo
<details>
    <summary>Code</summary>

    ```
    - name: add sudoers file for siko
    tags: always
    copy:
      src: sudoer_siko
      dest: /etc/sudoers.d/siko
      owner: root
      group: root
      mode: 0040
    ```

</details>
4. Now add `remote_user=siko` to your config file (ansible.cfg). <br>
Now we can skip typing `--ask-become-pass` in our command and shorten it to: `ansible-playbook site.yml`. <br>
5. Now copy your `site.yml` file and create a `bootstrap.yml` file. The bootstrap file will contain everything that is neccesary before we work on the nodes/hosts.<br>
Note: The code for that is `cp site.yml bootstrap.yml`.<br>
The bootstrap file should only contain the most basic things, including: creating a user, giving it sudoers and updating the host/node.<br>

### Roles
1. First create a backup of your `site.yml` file, since we are restructering everything.
2. We are now deleting everything from `site.yml` and only adding the Update Task Block and the initiation of roles:<br>
<details>
    <summary>Code</summary>

```
 hosts: all
  become: true
  pre_tasks:

  - name: update repository index (CentOS)
    tags: always
    dnf:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "CentOS"

  - name: update repository index (Debian)
    tags: always
    apt:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "Debian"

- hosts: all
  become: true
  roles:
    - base
```

</details>
3. Now we need to create the folder structure for the roles, just type:<br>
`mkdir roles` and in that folder `mkdir base`and in that folder `mkdir tasks` and in the task-folder you create your first "Taskbook" `main.yml`.<br>
A taskbook does not need `hosts:` or stuff like that and can only contain tasks:
```
- name: add ssh key for siko
    authorized_key:
      user: siko
      key: "< PUBLIC SSH-KEY >"
```


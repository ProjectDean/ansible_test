# ansible_test

Set up SSH Keys:

1. Copy personal SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>
2. Create ansible SSH Key
    ssh-keygen -t ed25519 -C "ansible" || (-t stands for type, -C stands for comment)<br>
    Note: rename the key to "ansible"<br>
3. Copy ansible SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP><br>
4. Making sure OpenSSH uses the right key:
    ssh -i ~/.ssh/ansible <SERVER-UP><br>

Set up Git:

1. Create a Respository
2. git clone <Git-Link>
3. Edit files
4. git add <Edited File>
5. git commit -m "commit-comment, what you have done"

Starting with Ansible:

1. First install ansible however you like
2. Create your first inventory:
    This file contains a list of your managed nodes/hosts, either their DNS or their IP

3. First Ansible command:
    ansible all --key-file ~/.ssh/ansible -i inventory -m ping
        "--key-file"" is the ssh key that is used
        "inventory" is the file with the IP addresses
        "-m ping" is the module that is used
4. Creating an ansible.cfg file to shorten the command:
    [defaults]
    inventory = inventory
    private_key_file = ~/.ssh/ansible
5. Now you can leave out keyfile and inventory:
    ansible all -m ping
6. Another command to see all managed nodes/hosts:
    ansible all --list-hosts
7. Another command, this time a bit more useful:
    ansible all -m gather_facts
        you basically see EVERYTHING of the servers
        --limit <IP-ADDRESS> limits it to the one server






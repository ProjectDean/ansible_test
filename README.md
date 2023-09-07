# ansible_test

Set up SSH Keys:

1. Copy personal SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>
2. Create ansible SSH Key
    ssh-keygen -t ed25519 -C "ansible" || (-t stands for type, -C stands for comment)
    Note: rename the key to "ansible"
3. Copy ansible SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>
4. Making sure OpenSSH uses the right key:
    ssh -i ~/.ssh/ansible <SERVER-UP>

Set up Git:

1. Create a Respository
2. git clone <Git-Link>
3. Edit files
4. git add <Edited File>
5. git commit -m "commit-comment, what you have done"

Starting with Ansible:

1. First install ansible however you like
2. Create your first inventory:
    This file contains a list of your managed notes, either their DNS or their IP





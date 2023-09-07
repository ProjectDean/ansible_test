# ansible_test

1. Copy personal SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>
2. Create ansible SSH Key
    ssh-keygen -t ed25519 -C "ansible" || (-t stands for type, -C stands for comment)
    Note: rename the key to "ansible"
3. Copy ansible SSH Key to the server
    ss-copy-id -i ~/.ssh/SSHKEY.pub <SERVER-IP>
4. Making sure OpenSSH uses the right key:
    ssh -i ~/.ssh/ansible <SERVER-UP>



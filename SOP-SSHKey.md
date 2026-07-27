# Set Up SSH Key Authentication on a Linux Server

### IT Guys
111 4th Ave

431-733-4051

Revesion info: Version 1.0

Approval Table:

## **Purpose**

This document outlines the standard procedures to enable secure and reliable remote administration of Linux servers using SSH with key-based authentication.

## **Audience**

This document is for systems administrators, network administrators, or IT professionals who are authorized to access a Linux server.

## **Scope**

This SOP explains how to:

1. Prepare a Linux server for remote administration
2. Install and verify the SSH service
3. Generate an SSH key pair on a client computer
4. Transfer the public key to the server
5. Configure the Linux user account for key-based access
6. Connect to the server using the private key
7. Verify that SSH key authentication works
8. Configure basic firewall access for SSH
9. Troubleshoot common connection and permission problems
10. Apply basic SSH security practices

## **Prerequisites**

- A Linux Server
- A workstation with a Linux OS to be used by the administrator
- A network connection between the Linux server and the workstation

## **Definitions**

- SSH (Secure Shell) - A cryptographic protocol that provides secure remote login and command execution over an unsecured network.
- OpenSSH Server — The Linux service that implements SSH, allowing remote clients to connect securely.
- SSH Key Pair — A matched set of cryptographic keys (public and private) used for secure, passwordless authentication.
- Public Key — The shareable half of an SSH key pair; placed on the server to authorize access.
- Private Key — The confidential half of an SSH key pair; stored securely on the client and never shared.
- Authorized Keys File — A server-side file (~/.ssh/authorized_keys) containing public keys allowed to authenticate to a user account.
- Firewall — A network security system that controls incoming and outgoing traffic based on predefined rules.
- UFW (Uncomplicated Firewall) — A simplified firewall management tool commonly used on Ubuntu/Debian systems.
- firewalld — A dynamic firewall manager used on RHEL/CentOS systems.
- SELinux (Security-Enhanced Linux) — A security module providing mandatory access controls; can affect SSH permissions.
- SSH Daemon (sshd) — The background service on the server that listens for and handles SSH connections.
- Key-Based Authentication — A login method using cryptographic keys instead of passwords for stronger security.
- Port 22 — The default network port used by SSH for incoming connections.
- Fail2ban — A security tool that blocks IP addresses after repeated failed login attempts to prevent brute-force attacks.

## **Procedure Steps**

### 1. Preparing the Linux Server for Remote Administration
- Ensure the server has a static IP or DNS entry
- Confirm administrative access
- Update system packages:
  
  `sudo apt update && sudo apt upgrade -y`

- Set hostname for the server
  - Open terminal then edit the */etc/hosts* file with a text editor of your choice, this example is using the *nano* command.
    
    `sudo nano /etc/hosts`

  - In */etc/hosts*, look for the *localhost* entry and update it to use your new hostname and the IP address of your machine. If there isn't a *localhost* entry, add one on a new line. Then save the file.

    `*server-ip-address*  localhost  *your-new-hostname*`

    Note: Replace *server-ip-address* with the actual static IP address of the server. Replace *your-new-hostname* to the desired hostname of the server.

  - Set your new hostname by running the *hostnamectl* command with your new hostname.
 
    `sudo hostnamectl set-hostname *your-new-hostname*`

    Note: Replace *your-new-hostname* to the desired hostname of the server.

- Create an administrative user:

  `sudo adduser *username*`

  `sudo usemod -aG sudo *username*`

  Note: Replace the *username* with the actual desired username.

### 2. Install and Verify the SSH Service
- Install OpenSSh Server:

  `sudo apt install openssh-server -y`
- Verify sevice status:

  `sudo systemctl status ssh`
- Enable SSH at boot:

  `sudo systemctl enable ssh`

### 3. Generate an SSH Key Pair on the Client
- On the administrator's workstation:

  `ssh-keygen`

  Note: By default this will create a 3072-bit RSA key pair
  
- Key locations:
  - Private key: `~/.ssh/id_rsa`
  - Public key: `~/.ssh/id_rsa.pub`
 
  <img width="744" height="307" alt="sshkeygen" src="https://github.com/user-attachments/assets/16136d5c-c057-4deb-8676-578b9491c4a8" />

- If you had previously generated an SSH key pair, you may see a prompt that looks like this:

  <img width="747" height="172" alt="overwrite" src="https://github.com/user-attachments/assets/2335ddd1-e8fa-400e-9765-9e0ac1efb6ee" />

- If you choose to overwrite the key on disk, you will not be able to authenticate using the previous key anymore.
- Next, you will be prompted to enter a passphrase for the key. This is an optional passphrase that can be used to encrypt the private key file on disk. If you enter one, you will have to provide it every time you use this key. It is recommended to use a passphrase, but if not needed, you can press *enter* to bypass this prompt.

  <img width="742" height="187" alt="passphrase" src="https://github.com/user-attachments/assets/6a602828-7d45-48ff-a04d-70e505ab2047" />

- The image below is an example that shows you now have a public and private key that you can use to authenticate.

### 4. transfer the Public Key to the Server
  `ssh-copy-id *username*@*server-ip*`  If using the IP address of the server. Replace *server-ip* with the actual IP address, and the *username* with the username made from the previous step.

  or

  `ssh-copy-id *usename*@*hostname*`  If using the hostname set from previous step. Replace *hostname* with the configured hostname, and the *username* with the username made from the previous step.

- You may see a message like this:
  
  <img width="736" height="186" alt="continue" src="https://github.com/user-attachments/assets/989a5f29-ec51-4352-9134-f51baa64614e" />

  - This means that your local computer does not recognize the remote host. This will happen the first time you connect to a new host. Type *yes* and press *enter* to continue.
 
- Enter the password of the remote user's account, then press *enter*. You will see an output that looks like this:
  
  <img width="733" height="198" alt="keyadded" src="https://github.com/user-attachments/assets/600ccb71-3912-4741-bb78-22c38915cf39" />

### 5. Configure the Linux User Account for Key-Based Access
On the server:
  `mkdir -p ~/.ssh`
  
  `chmod 700 ~/.ssh`
  
  `cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`
  
  `chmod 600 ~/.ssh/authorized_keys`
  
  `rm ~/id_rsa.pub`

### 6. Connect to the Server Using the Private Key
- From the client:

  `ssh *username*@*server-ip*` or `ssh *username*@*hostname*`

  Note: Replace *username* with the username made from the previous step. Replace *server-ip* with the IP address of the server or replace *hostname* with the configured hostname.

### 7. Verify SSH Key Authentication
- Confirm login works without a password prompt.
- Check server logs:

  `sudo journalctl -u ssh`

- Confirm the session shows *publickey* authentication:

  `sudo grep "Accepted publickey" /var/log/auth.log`

### 8. Configure Basic Firewall Access for SSH
- For Ubuntu/Debian:

  `sudo ufw allow ssh`

  `sudo ufw enable`

  `sudo ufw status`

- For RHEL/CentOS:

  `sudo firewall-cmd --permanent --add-service=ssh`
  
  `sudo firewall-cmd --reload`

### 9. Troubleshoot Common Connection and Permission Problems
- Permission denied (public key)
  - Incorrect file permissions
    `chmod 700 ~/.ssh`

    `chmod 600 ~/.ssh/authorized_keys`

  - Wrong username or homw directory
  - Key not copied correctly

- SSH service not running

  `sudo systemctl restart ssh`

- Firewall blocking port 22
  - Ensure rule allow SSH.
- SELinux issues (Rhel/CentOS)

  `sudo restorecon -R -v ~/.ssh`

### 10. Apply Basic SSH Security Practices
- Disable password authentication, edit */etc/ssh/sshd_config*:
  
  `PasswordAuthentication no`

- Disable root login:

  `PermitRootLogin no`

- Optional, change default SSH port:

  `Port 2222`

- Enable automatic security updates.
- Use fail2ban to block brute-force attempts:

  `sudo apt install fail2ban -y`

## **References**

- https://docs.digitalocean.com/products/paperspace/machines/how-to/manage-hosts/
- https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server

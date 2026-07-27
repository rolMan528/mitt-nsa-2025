# Set Up SSH Key Authentication on a Linux Server

### IT Guys
111 4th Ave

431-733-4051

Revesion info: Version 1.0

Approval Table:

## **Purpose**

This document outlines the standard procedures to enable secure and reliable remote admionistration of Linux servers using SSH with key-based authentication.

## **Scope**

This SOP explains how to:

1. Prepare a Linux server for remote administration
2. Install and verify the SSH service
3. Genrate an SSH key pair on a client computer
4. Transfer the public key to the server
5. Configure the Linux user account for key-based access
6. Connect to the server using the private key
7. Verify that SSH key authentication works
8. Configure basic firewall access for SSH
9. Troubleshoot common connection and permission problems
10. Apply basic SSH security practices

## **Accountability Matrix**

## **Definitions**

## **Procedure Steps**

### 1. Preparing the Linux Server for Remote Administration
- Ensure hte server has a static IP or DNS entry
- Confirm administrative access
- Update system packages:
  
  `sudo apt update && sudo apt upgrade -y`
- Create an administrative user:

  `sudo adduser adminuser`

  `sudo usemod -aG sudo adminuser`

### 2. Install and Verify the SSH Service
- Install OpenSSh Server:

  `sudo apt install openssh-server -y`
- Verify sevice status:

  `sudo systemctl status ssh`
- Enable SSH at boot:

  `sudo systemctl enable ssh`

### 3. Generate an SSH Key Pair on the Client
- On the administrator's workstation:

  `ssh-keygen -t ed25519 -C "adminuser@server"`
- Key locations:
  - Private key: `~/.ssh/id_ed25519`
  - Public key: `~/.ssh/id_ed25519.pub`

### 4. transfer the Public Key to the Server
  `ssh-copy-id adminuser@*server-ip*`

Note: Replace *server-ip* with the actual IP address of the server.

### 5. Configure the Linux User Account for Key-Based Acccess
On the server:
  `mkdir -p ~/.ssh`
  
  `chmod 700 ~/.ssh`
  
  `cat ~/id_ed25519.pub >> ~/.ssh/authorized_keys`
  
  `chmod 600 ~/.ssh/authorized_keys`
  
  `rm ~/id_ed25519.pub`

### 6. Connect to the Server Using the Private Key
- From the client:

  `ssh admin@*server-ip*`

Note: Replace *server-ip* with the actual IP address of the server.

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
    `chmod 700



# The-ssh-protocol
I'll show you how to set up the ssh protocol, configure fail2ban and set up wireshark on the remote server on port 2222.

# Table of Contents
- [1. Introduction to SSH and Remote Access](#1-introduction-to-ssh-and-remote-access)
- [2. SSH Key Authentication](#2-ssh-key-authentication)
- [3. File Transfer with SCP and SFTP](#3-file-transfer-with-scp-and-sftp)
- [4. Creating SSH tunnels and port forwarding](#4-creating-ssh-tunnels-and-port-forwarding)
  - [4.1. Activate the ionos Port on the ionos customer site](#41-activate-the-ionos-port-on-the-ionos-customer-site)
  - [4.2. Setting up the UFW firewall](#42-setting-up-the-ufw-firewall)
  - [4.3. Modified the remote server config file](#43-modified-the-remote-server-config-file)
- [5. Securing your SSH server with Fail2ban](#5-securing-your-ssh-server-with-fail2ban)
- [6. Monitoring network traffic with Wireshark](#6-monitoring-network-traffic-with-wireshark)
  - [6.1. Installation of Wireshark](#61-installation-of-wireshark)
  - [6.2. Capture packets on our remote server](#62-capture-packets-on-our-remote-server)
  - [6.3. Send the log file to our local machine](#63-send-the-log-file-to-our-local-machine)
  - [6.4. Open Wireshark on our local machine to read the log file](#64-open-wireshark-on-our-local-machine-to-read-the-log-file)
- [7. Theoretical analysis](#7-theoretical-analysis)
  - [7.1. Data confidentiality](#71-data-confidentiality)
  - [7.2. Changing ports and using Fail2Ban](#72-changing-ports-and-using-fail2ban)
  - [7.3. Observation with Wireshark](#73-observation-with-wireshark)
- [8. Conclusion](#8-conclusion)
- [9. Resources](#9-resources)

## 1.Introduction to SSH and Remote Access
<p>To start with, I'm going to connect to my ssh server using an ionos vps remote server running linux unbutu.</p><p>To do this, I'm going to use the ssh command :<br>ssh root@ipadress</p><p>We need to connect to root to create a user, to do this we need to type a command: <p>adduser username</p><p>When we have created our user, we connect to it and it will ask us to enter the password we have created for the user:</p><p>ssh username@ipadress<p>username@ipadress's password:</p>

## 2.SSH Key Authentication
<p>Once the user has been created, we go back to the local machine and make an SSH key connection using the command :</p><p>ssh-keygen -t ed25519</p><p>The key is in two parts: the private key, which is encrypted and must not be shared, and the public key, which can be shared to decrypt the private key.</p><p>The private key is stored in ~/.ssh/id_rsa and the public key in
~/.ssh/id_rsa.pub.</p><p>Once our key has been created, we'll copy the public key to our remote server and connect without the password prompt using this command:</p><p>ssh-copy-id username@ipadress</p><p>There is still one step to connect to the key without requesting the password, we must access our file on the remote server:</p><p>sudo nano /etc/ssh/sshd_config</p><p>Once in this file, the line is modified:</p><p>PasswordAuthentication no</p><p>Once done, restart the remote server with this command:</p><p>sudo systemctl restart ssh</p><p>We then test this connection with the user with whom we created the key:</p><p>ssh username@ipadress</p>

## 3.File Transfer with SCP and SFTP
<p>We're going to transfer a file using SCP and SFTP to our remote server, starting by going back to our local machine to create a txt file:</p><p>echo "This is a test file" > file.txt</p><p>We will then transfer this file to the remote server:</p><p>scp file.txt username@ipadress:/home/username/</p><p>Next, we'll check whether the file is present on the remote server:</p><p>We will then use SFTP, which, unlike SCP, allows interrupted file transfers to be resumed. Error and warning management: SFTP offers built-in mechanisms for handling errors and warnings, whereas SCP does not, with this command, we access the SFTP service:</p><p>sftp username@ipadress</p><p>Next, we want to browse remote directories:</p><p>ls</p><p>cd /home/username/</p><p>Then we transfer the file to the local machine using SFTP:</p><p>put file.txt</p>

## 4.Creating SSH tunnels and port forwarding

### 4.1.Activate the ionos Port on the ionos customer site
<p>To set our Port 2222, we need to access our server on the ionos site and access the networks tab on the left, the firewall strategy sub-section once inside, we open by clicking on the strategy, we see different ports, we add port 2222 and we validate the server will relaunch its firewall, taking 2222 into account, but that's not all to connect to port 2222.</p>

### 4.2.Setting up the UFW firewall
<p>To set up our Port 2222, we need to set up the UFW firewall, which needs to be activated and configured. I'll explain how to do this using the various commands to set up UFW with Port 2222:</p><p>We check whether UFW is installed on our server:</p><p>sudo apt install ufw</p><p>Policies are put in place by default:</p><p>sudo ufw default deny incoming</p><p>sudo ufw default allow outgoing</p><p>SSH connections are authorised:</p><p>sudo ufw allow ssh</p><p>sudo ufw allow 22</p><p>sudo ufw allow 2222</p><p>You can authorise other tcp connections by putting back the same line:</p><p>sudo ufw allow 22/tcp</p><p>sudo ufw allow 2222/tcp</p><p>Finally, for UFW, we've just placed the order for active:</p><p>sudo ufw enable</p>

### 4.3.Modified the remote server config file
<p>After configuring UFW, add Port 2222 to the server config file. I recommend keeping Port 22 to have a backup port:</p><p>sudo nano /etc/ssh/sshd_config</p><p>Then restart the ssh system with this command:</p><p>sudo systemctl restart ssh</p><p>In order for ionos to take Port 2222 into account, we need to reinstall the openssh-server:</p><p>sudo apt reinstall openssh-server</p><p>We redo a small systemctl restart ssh.</p><p>Then we return to our local session and try to connect to our port 2222 using this command:</p><p>ssh -p 2222 username@ipadress or ssh username@ipadress -p 2222</p>

## 5.Securing your SSH server with Fail2ban
<p>We're going to configure Fail2ban on our server to protect against a Bruteforce attack by blocking IPs after several failed attempts:</p><p>sudo apt install fail2ban</p><p>You need to configure Fail2ban on your file:</p><p>sudo nano /etc/fail2ban/jail.local</p><p>The section is added to the file:</p><p>[sshd]</p><p>enabled = true</p><p>port = 2222</p><p>Then restart Fail2ban:</p><p>sudo systemctl restart fail2ban</p>

## 6.Monitoring network traffic with Wireshark
<p>Wireshark aims to visually show the encrypted exchanges between a client and a server during an SSH connection.</p>

### 6.1.Installation of Wireshark
<p>Wireshark aims to visually show the encrypted exchanges between a client and a server during an SSH connection.</p><p>We're going to install wireshark on our local machine and on our remote server. On our local machine, we'll be able to use wireshark's graphical interface to analyse the logs from our remote server, which we won't be able to use the graphical interface, we'll have to use tshark and send the log file we've saved on the server to our local machine.</p><p>The command to install wireshark is:</p><p>sudo apt install wireshark</p>

### 6.2.Capture packets on our remote server
<p>To capture the packets on our remote server with wireshark, we don't have a graphical interface, so we need to use tshark we need to use this command:</p><p>sudo tshark -i ens6 -w /tmp/capture_ssh.pcap
</p><p>If you want to add a filter to capture only port 2222, for example, you need to enter this command:</p><p>sudo tshark -i ens6 -f "tcp port 22" -w /tmp/capture_ssh.pcap</p>

### 6.3.Send the log file to our local machine
<p>On our terminal on our local machine, we're going to use scp to send our file to our local machine, using this command:</p><p>scp your_username@your_server_ip:/tmp/capture_ssh.pcap ~/Desktop/</p><p>your_username with your user name on the VPS server.</p><p>your_server_ip by the IP address of your VPS server.</p><p>~/Desktop/ by the path where you want to save the file on your local machine (you can choose another directory if you wish).</p>

###6.4.Open Wireshark on our local machine to read the log file
<p>On the terminal of our local machine, we will open Wireshark with this command:</p><p>sudo wireshark</p><p>Once Wireshark is open, click File at the top left, then Open. This will open a window to select our log file, then click open and Wireshark will return to the main window with the file open, you will see SYN and ACK packets which are used to establish a TCP connection, as well as SSH packets, these packets show that the SSH connection has been successfully established, but the contents of the packets are encrypted, so unreadable.</p>

## 7.Theoretical analysis
<p>Encrypting SSH connections is crucial for guaranteeing the confidentiality and integrity of data exchanged between the client and the server. Here are the main reasons why this encryption is necessary, as well as the impact of additional measures such as port switching or the use of Fail2Ban.</p>

### 7.1.Data confidentiality
<p>SSH (Secure Shell) is a secure protocol used to establish encrypted communication between a client and a server. Encryption is essential to protect sensitive data (passwords, commands, files) from eavesdropping on the network.
Without encryption, an attacker could intercept network traffic and gain access to the information exchanged, including identification details and commands executed. With encryption, even if the traffic is captured, the attacker cannot decrypt the content without the encryption key.</p>

### 7.2.Changing ports and using Fail2Ban
<p>Changing the port:</p><p>Changing the default SSH port (port 22) makes automated attacks more difficult, as most brute force attacks target this port by default. This does not directly strengthen cryptographic security, but it does reduce the visibility of the SSH service for non-targeted attackers.</p><p>Fail2Ban:</p><p>Fail2Ban is a tool that monitors access logs for suspicious connection attempts (e.g. multiple failed connections). If such activity is detected, Fail2Ban can temporarily ban the source IP, limiting the window of opportunity for a brute force attack. This strengthens security by limiting the chances of compromise by automated attacks.</p>

### 7.3.Observation with Wireshark
<p>Wireshark, a packet analyser, can observe packet exchanges on the network, including those linked to SSH. However, although Wireshark can capture SSH packets, the content of the exchanges remains encrypted and therefore unreadable.
However, by observing packet metadata (such as the source and destination IP address, the port used, the TCP protocol, and the timing of the packets), an attacker could:</p><p>Deduce information about server usage patterns (for example, which IP connects frequently).
Identify whether the standard SSH port (22) is being used, facilitating brute force attack attempts.</p><p>The SSL/TLS encryption used by SSH protects the confidentiality and integrity of data, but unencrypted information, such as packet headers, can provide clues to an observer.</p>

## 8.Conclusion
<p>The SSH protocol plays a fundamental role in securing communications on unsecured networks such as the Internet. Thanks to its robust encryption, it protects sensitive data exchanged between a client and a server, guaranteeing both confidentiality and integrity. However, the security of an SSH connection does not rely solely on encryption. Additional measures such as changing the default port and using tools such as Fail2Ban help to strengthen protection by reducing the risk of automated and brute force attacks. Although observing network flows using tools such as Wireshark cannot directly decrypt encrypted data, it can provide information that can be exploited by attackers. It is therefore essential to combine the encryption of SSH connections with hardening strategies, in order to minimise attack surfaces and strengthen the overall security of systems.</p>

## 9.Resources
<p>ChatGPT</p><p>Tutorial for configuring UFW: https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server</p>

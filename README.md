# The-ssh-protocol
I'll show you how to set up the ssh protocol, configure fail2ban and set up wireshark on the remote server on port 2222.

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
<p>Wireshark aims to visually show the encrypted exchanges between a client and a server during an SSH connection.</p><p></p>

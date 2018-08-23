# linuxServerConfiguration
deployment of CatalogApp project 

# Description:

Baseline installation of a Linux server and prepare it to host your web applications here its catalogApp. Secure server from a number of attack vectors, install and configure a database server, and deploy catalog applications onto it.

# Important info
IP address: 18.222.164.3

Accessible SSH port: 2200.
Application URL: http://18.222.164.3.xip.io/

# Steps to configure Linux Server
## Create a new user named grader and grant this user sudo permissions.
1. Log into the remote VM as root user through ssh: $ ssh root@18.222.164.3.
2. Add a new user called grader: $ sudo adduser grader.
3. Create a new file under the suoders directory: $ sudo nano /etc/sudoers.d/grader. 
4. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.

## Update all currently installed packages
1. Type and run  $ sudo apt-get update.
2. Type and run $ sudo apt-get upgrade.
3. Install finger, a utility software to check users' status: $ apt-get install finger.

## Configure the key-based authentication for grader user
1. Generate an encryption key on your local machine with: $ ssh-keygen -f ~/.ssh/udacity_key.rsa.
2. Log into the remote VM as root user through ssh and create the following file: $ touch /home/grader/.ssh/authorized_keys.
3. Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. 
4. Then change some permissions:
      i. $ sudo chmod 700 /home/grader/.ssh.
      ii. $ sudo chmod 644 /home/grader/.ssh/authorized_keys.
5. Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh.
6. Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/udacity_key.rsa grader@18.222.164.3

Server Tutorial
===============

Instructions for a Mac running a remote AWS EC2 server (Ubuntu 16.04.1 LTS xenial). Approximately 21 Nov 2016 - 30 Dec 2016.



-----
[1]: aws.amazon.com/free
[2]: https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/
[3]: https://www.digitalocean.com/community/tutorials/additional-recommended-steps-for-new-ubuntu-14-04-servers
[3]: https://alestic.com/2009/04/ubuntu-ec2-sudo-ssh-rsync/
[4]: http://askubuntu.com/questions/192050
[5]: https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04 "DigitalOcean Linux/Nginx/MySQL on Ubuntu"


AWS
----
Sign up for an [AWS free account][1]. Launch an AWS [EC2 Instance][2]. Set up security groups. Note: if IPv6 (AWS doesn't know), google "what is my ipv4 ip" (ask Andy). Think about making users?

Assign an Elastic IP to your running instance. Let's say it's

    22.22.222.222

Work on this part :)

Get a Text Editor
----
I recommend Textmate. Get it. Instructions how. Setup rmate.

Terminal Basics
----
* What is Bash shell
* ls -la
* cd
* mkdir
* rm -rf (recursive removal)
* tilde ~/
* tabbing between terminal tabs
* new tab `Command-T`
* check moita.sh

SSH
----
Open the Terminal app. Locate your private key (a .pem file). Let's say it's located at

    /../AWS_key.pem

Protect the file.

    chmod 600 /../AWS_key.pem

Now, we're going to define a shortcut command (an `alias`) to ssh into your remote server. Let's call it `sshAWS`. To make your alias, you'll have to edit two hidden files on your local computer. To edit a file from Terminal, you can run `mate /path/to/file`.

1. Edit ~/.bashrc and add the line. Be careful, there CANNOT be a space between your alias name, the equals sign, and the first quotation, because your Terminal app's Bash shell will parse that as separate, nonsensical commands. 

    `alias sshAWS="ssh -R 52698:localhost:52698 -i /../AWS_key.pem ubuntu@22.22.222.222"`

2. Edit ~/.bash_profile and add the following line at the top of the file. This is important because all Terminal bash shells call ~/.bash_profile.

    `source ~/.bashrc`

Now, you can run this in Terminal to propagate changes.

    source ~/.bash_profile

Excellent! SSH login as the default user `ubuntu`:

    sshAWS

Now you are in the remote server command line. To update packages, run

    sudo apt-get update && sudo apt-get upgrade

The command line will prompt you with (y/n) question to agree to changes.

    y

To leave, try:

    exit


Firewall and other short setup thingys
----
Related to your EC2 Security Groups. We'll open the ports on the remote-server side now![3] SSH into your remote server.

    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw allow 52698/tcp
    sudo ufw show added
    sudo ufw enable
    y

Configure timezones and ntp global timing

    sudo dpkg-reconfigure tzdata
    sudo apt-get update
    sudo apt-get install ntp

Swap. Check the tutorial[3]. Check if you have an SSD, using swap?

    swapon -s
    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'

Users
----
The default user, `ubuntu`, has passwordless `sudo` privileges.[4] All other users you create will need a password.[5] 

Just so you can switch users, you should set `ubuntu`'s password.

    sudo passwd ubuntu

and enter your password twice.

Let's set up a test user.

    sudo adduser testuser

Enter `testuser`'s password twice. Let's say it's

    pwtestuser

Enter a few times to exit the screen. You've now created another user!

For `testuser` to have `sudo` privileges, as `ubuntu`, try

    usermod -aG sudo testuser

To switch users, 

    su - testuser

and put in your password.

Now, let's say you want to lighten your security by not requiring a `sudo` password for new users. See [4] and [5].

Finally, disable root login from ssh.[6]

    sudo nano /etc/ssh/sshd_config

Uncomment and make sure this line exists:

    /etc/ssh/sshd_config

Save the file, then restart ssh.

    service ssh restart

You'll need your password. Logout!

    exit


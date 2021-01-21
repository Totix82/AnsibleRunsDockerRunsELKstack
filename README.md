# AnsibleRunsDockerRunsELKstack

1) Install VirtualBox for two Ubuntu Virtual Machine (one for the Ansible Controller and one for the oorchestrated VM)

Install VirtualBox (X OS version for Max)
Run VirtualBox

Create two Virtual Machines

Click 'New' button to open a dialog. Create virtual machine

Type a name for the new virtual machine. Since I am planning to install Ubuntu 14.04, I'll enter 'ubuntu1404'. Note that VirtualBox automatically changes 'Type' to Linux and 'Version' to 'Ubuntu (64 bit)'. These two options are exactly what we need. Name virtual machine

The memory size depends on your host machine memory size. In my case, I have 12GB physical RAM. I like to allocate as much as possible for Ubuntu but leave some for my Windows host machine. I pick 8192 MB for my Ubuntu. Note that VirtualBox will create a swap partition with the same amount space as base memeory you have entered here. So later when you are selecting the size of the virtual hard drive, make sure it is large enough since the hard drive will be splitted into root (/)and swap partitions. The root partition contains by default all your system files, program settings and documents.

Control memory size

Accept the default 'Create a virtual hard drive now' and click 'Create' button. Create virtual drive

Continue to accept the default 'VDI' drive file type and click 'Next' button. Select hard drive type

Change the storage type from the default 'Dynamically allocated' to 'Fixed size' to increase performance. Select storage type

For the virtual hard drive space, the default value is 8GB which is too little for RNA-Seq analysis. I'll pick 100GB since I have plenty of space in my hard disk. You want to choose a good size for your RNA-Seq analysis. If you realize the drive space is not large enough, you'll need to go over these steps again to create another virtual machine.

Confirm virtual drive settings

Click 'Create' button and VirtualBox will generate Ubuntu virtual machine. Create a virtual drive

Now the virtual machine is created. We are ready to install Ubuntu in this virtual machine. Select your new virtual machine and click 'Settings' button. Click on 'Storage' category and then 'Empty' under Controller:IDE. Click "CD/DVD" icon on right hand side and select the ubuntu ISO file to mount.

Note that if you have not downloaded 64-bit Ubuntu ISO file, you can check out this page for more information. When downloading Ubuntu ISO file, make sure to selecte 64-bit version. Also make sure the VT-x/Virtualization Technology has been enabled in your computer's BIOS/Basic Input Output System. Select IDE Select an image file Done with selecting the image

Notice that if OK button is greyed out when going into settings, then over into the error message to find out the issue. For example it could be that the CPU is too high to support the IDE boot.

Since Tophat program can take an advantage of multiple processors/threads, it is a good idea to specify a large number of processors in virtual machine (default value is 1). You can change this number by clicking on 'System' category. In this case, I change the number of CPUs to 4 since 4 is the largest value shown on the green bar in my case. Now you can click 'OK' button to continue.
Select processor number

VirtualBox may pop up a message about 'Auto capture keyboard' option. Read the message there and check 'Do not show this message again' option before clicking OK.

Capture keyboard
Install Ubuntu

Back to Oracle VM VirtualBox Manager, click on the new Ubuntu virtual machine and hit 'Start' button. Now you shall see a 'Welcome' screen. Click 'Install Ubuntu' button. Note that the installation process may differ a little bit from version to version. The screenshots here are based on Ubuntu 14.04.1. First screen in installing Ubuntu

Click 'Continue' button. Prepare to install Ubuntu

Make sure 'Erase disk and install Ubuntu' option is selected and click 'Install Now' button. Confirm installation type

Ubuntu will ask you a few questions. If the default is good, click 'Continue' button. Choose geolocation Keyboard layout

In 'Who are you?' dialog, enter your preferred name, username and password. Note that this user will have root/sudo privilege. Click 'Continue' button. Who you are

The installation will continue until it is finished. Installing the system

After installation is complete, click 'Restart Now' button. When you see a screen with a black background saying 'Please remove installation media and close the tray (if any) then press ENTER:', just follow it. Restart Ubuntu

Enter the password you have chosen and press 'Enter'. Enter the password

The Ubuntu Desktop OS is ready. You may find the desktop screen is too small. Don't worry. You can solve this easily with "VirtualBox Guest Additions".


2) Install Ansible

Run the below commands
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
ansible --version
Next, we need to make it possible for our node to access the Ansible server. We do this via Secure Shell (SSH). Copy the server's SSH public key to the node. If your server doesn't have a key yet, generate one with the command:

ssh-keygen

You will be asked for a file name (keep the default) and to create/verify a passphrase for the key 

Creating an SSH key on the Ansible server.

Display the contents of the public SSH key with the command:

cat ~/.ssh/id_rsa.pub

Here's what you do with the output of that command:

    Copy the text from the key.
    Log into your node server.
    Issue the command sudo -s.
    Open the authorized_keys file with the command sudo nano ~/.ssh/authorized_keys.
    Paste the contents of the server key at the bottom of this file.
    Save and close the file.

If you want to simplify this process, issue the command (from the Ansible server):

ssh-copy-id NODE_IP

Where NODE_IP is the IP Address of the node to be added.

To test the newly added key, go back to your Ansible server and SSH to the node machine. Instead of being prompted for the user's password, you should be prompted for the SSH key passphrase.

3) Set up Ansible and Playbook to orchestrate Docker on a remote server, on which we will run a docker-compose file

cd /etc/Ansible

nano ansible.cfg

and uncomment, in the defaults, line

inventory = /etc/ansible/hosts

once pointed to it, add one or more remote IPs  

nano hosts

and add the following group:

[DockerServers]
192.168.99.103


Where SERVER_IP is the IP address of the Guest OS server. Save and close the file.

We're going to create a playbook that will do the following:

    Install aptitude
    Install a number of dependencies
    Add a docker repository
    Install docker-ce
    Install the docker Python module
    Install docker-compose
    copy docker-compose.yml file to Guest OS
    Enable Docker service
    Run ELK Stack docker-compose file

Tasks are pretty intutive and named with all of the above steps.

See file ubuntu_playbook.yml

Regarding the Docker file, see file docker-compose.yml

We can now run our Ansible playbook. To do this, issue the command:

ansible-playbook ubuntu_playbook.yml

Docker will create 3 Elasticsearch containers (in a cluster), 1 for Logstash and 1 for Kibana.

All of the containers will run on 256MB of RAM.

All of them will run on a network group called elastic

Process to build ansible webserver.  1 controller and 2 hosts

Guide:
https://drive.google.com/file/d/1UHOMwQuvnIbCeSrEDeV6CO6S2ofa5Cs9/view
vagrant installation:
https://cloudspinx.com/installation-of-vagrant-on-ubuntu/

note: we also need to install virtualbox
To remove virtualbox (during issues)
sudo apt-get remove virtualbox-\*
sudo apt-get purge virtualbox-\*

Part A
Ansible Controller Machine Setup
Ansible Hands-On | Step by Step for Beginners.pdf - Google Drive

Step 1 - Install VirtualBox and Vagrant on your local machine.
Step 2 - Open a terminal and navigate to the directory where you want to set up your Ansible project.
Step 3 - Create a new directory for your Ansible controller VM by running the command mkdir ansible-controller
Step 4 - Navigate to the directory and create a new file called Vagrantfile by running the command  vagrant init centos/7
HashiCorp Cloud Platform
Step 5 - Edit the Vagrantfile and add the lines to the end of the file to provision Ansible on the VM
Lsb_release -a  to check ubuntu version

Vagrantfile for creating VM for Ansible Controller - note with this step indentation is crucial and you may need to use ruby formatter online

Vagrant.configure("2") do |config|
  config.vm.define "ansible-controller" do |controller|
    controller.vm.hostname = "controller"
    controller.vm.box = "centos/7"
    controller.vm.provision "shell", inline: <<-SHELL
      sudo yum install epel-release -y
      sudo yum install ansible -y
    SHELL
  end
end

Step 6 - Save & check its a valid vagrantfile  vagrant validate  Then run command  vagrant up to start the VM
Vagrant status
Step 7 - Once the VM is up and running, connect to it using SSH by running the command vagrant ssh
Check ansible is installed - ansible --version
Step 8 - Create a new directory for your Ansible project on the controller VM by running the command mkdir ansible-project
Step 9 - Navigate to the ansible-project directory and create a new file called hosts by running the command touch hosts
Step 10 - Create a new file called playbook.yml. This file will contain the tasks you want to perform on your managed hosts
As of now the hosts and the playbook file are empty
We will now create some host machines that will be controlled by the Ansible controller


Before we can continue, we must edit the Repo fukw for Centos7 to allow installation:
To fix centos repo issue
vi /etc/yum.repos.d/CentOS-Base.repo
How to Fix “Cannot find a valid baseurl for repo” in CentOS
Fix "Cannot find a valid baseurl for repo" in CentOS - DEV Community

# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
mirrorlist=http://vault.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
baseurl=http://vault.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://vault.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
baseurl=http://vault.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://vault.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
baseurl=http://vault.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://vault.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
baseurl=http://vault.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

Part B
Ansible Host Machines Setup

Step 1 - On terminal navigate to your Ansible Project folder
Step 2 - Create a new directory for your host machines by running the command mkdir host-machines
Step 3 - Navigate to host-machines directory and create a new Vagrantfile by running the command vagrant init centos/7
Step 4 - Edit the Vagrantfile and modify the following lines to set up two Vagrant machines:

Vagrantfile for creating VMs for Ansible Host

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  
  config.vm.define "web" do |web|
    web.vm.hostname = "web"
    web.vm.network "private_network", ip: "192.168.33.10"
  end
  
  config.vm.define "db" do |db|
    db.vm.hostname = "db"
    db.vm.network "private_network", ip: "192.168.33.11"
  end
  config.vm.network "forwarded_port", guest: 80, host: 8080, auto_correct: true
  config.vm.usable_port_range = (8000..9000)
end

Step 6 - Save & check its a valid vagrantfile  vagrant validate  Then run command  vagrant up to start the VM
Step 7 - Check the status of machines  vagrant status
Once the VMs are up, connect to them using SSH vagrant ssh ＜machine-name＞       e.g vagrant ssh web
This completes the process of setting up host machines

Part C
Making connection between controller and host machines

Step 1 - Make sure all machines are up and running
Step 2 - Run command ip addr on each machine and check they have IP addresses in the same range (e.g. 192.168.33.x).
Step 3 - On Controller machine run the command ssh-keygen to generate an SSH key pair
Step 4 - Goto ~/.ssh folder and check the public and private keys generated
Step 5 - Copy the public key to the host machines by running the command ssh-copy-id ＜user＞@＜host＞
For example, to copy the public key to the web machine, run the command ssh-copy-id vagrant@192.168.33.10

Can do this manually by copying the contents of the .pub file generated by ssh-keygen and pasting it into the ~/.ssh/authorized_keys file on the host machines

Step 6 - Test the SSH connection by running the ssh command with the IP address of the host machines-
For example:  ssh vagrant@192.168.33.10
   ssh vagrant@192.168.33.11
  

Part D - Adding host and playbook file on Controller and Run Playbook
Step 1 - Connect to the ansible controller machine using ssh vagrant ssh
<machine name>
Step 2 - Edit the hosts file (vi hosts) to add the IP addresses or hostnames of
the machines you want to control. For example:
Ansible hosts file
[webservers]
192.168.33.10
[dbservers]
192.168.33.11
Step 3 - Run the command ansible all -m ping -i hosts to test the connection
to the host machines
Step 4 - Now edit the playbook.yml file and add instructions for host machines
For e.g. to install the Apache web server on your webservers, you can use the
following playbook

Ansible Playbook to install and start Apache web server
- name: Install Apache web server
  hosts: dbservers
  become: true
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: latest
    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: true

Step 5 - You can now run the playbook on the managed hosts by running the
command ansible-playbook -i hosts playbook.yml
Step 6 - Once the playbook has run, you can access the web servers by
opening a web browser and navigating to the IP addresses of your webservers
![image](https://github.com/user-attachments/assets/95adca7d-8221-4cb9-8cfc-e7d27a740022)

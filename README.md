# Ansible Refactoring Static Assignments - Armstrong

## Ansible Refactoring and Static Assignments (Imports and Roles)

### Self study

https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html


# Refactoring Ansible code by importing other playbooks into site.yml


## Step 1 - Jenkins job enhancement

We will make some changes to our Jenkins job 

- now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. 

Besides, it consumes space on Jenkins serves with each subsequent change. 

We will enhance it by introducing a new Jenkins project/job 

- we will require ***Copy Artifact plugin***.

1. create a new directory in Jenkins-Ansible server called  

```
mkdir ansible-config-artifact
```

- we will store there all artifacts after each build.

```
sudo mkdir /home/ubuntu/ansible-config-artifact
```

![mkdir](images/mkdir.jpg)


2.  Change permissions to this directory, so Jenkins could save files there.

```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on ***Available*** tab search for ***Copy Artifact*** and install this plugin without restarting Jenkins

![pluggin](<images/copy artifact pluggin.jpg>)

![pluggin installed](<images/pluggin install.jpg>)

4. Create a new Freestyle project and name it ***save_artifacts***.

![new free style](<images/new free style.jpg>)


5. Configure This project accordingly so it will be triggered by completion of our existing ansible project. 


![new free style1](<images/new free style 1.jpg>)

![new free style2](<images/new free style 3.jpg>)

![new free style3](<images/new free style 4.jpg>)

NOTE : can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

6. The main idea of save_artifacts project is to save artifacts into ```/home/ubuntu/ansible-config-artifact``` directory. 

To achieve this, create a Build step and choose Copy artifacts from other project, specify ***ansible*** as a source project and ```/home/ubuntu/ansible-config-artifact``` as a target directory.

![target directory](<images/target directory.jpg>)


7. Test your set up by making some change in README.MD file inside your ***ansible-config-mgt*** repository (right inside ***master*** branch).

If both Jenkins jobs have completed one after another - you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.

Now your Jenkins pipeline is more neat and clean.

![edit repo](<images/edit repo.jpg>)

![edit repo2](<images/edit repo1.jpg>)

![edit repo3](<images/edit repo2.jpg>)

# Step 2 - Refactor Ansible code by importing other playbooks into site.yml

Before starting to refactor the codes:

1. Pull down the latest code from master (main) branch, and create a new branch, name it ***refactor***.

![refactor branh](<images/branch refactor.jpg>)

DevOps philosophy implies constant iterative improvement for better efficiency - refactoring is one of the techniques that can be used, but you always have an answer to question "why?". 

Why do we need to change something if it works well?

In Project 11 you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. In this case, you will have to read through the whole playbook to check if all tasks written there are applicable and is there anything that you need to add for certain server/OS families. 

Very fast it will become a tedious exercise and your playbook will become messy with many commented parts. Your DevOps colleagues will not appreciate such organization of your codes and it will be difficult for them to use your playbook.

Most Ansible users learn the one-file approach first. However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

## Let see code re-use in action by importing other playbooks.

1. Within playbooks folder, create a new file and name it ***site.yml*** 

![site yml](<images/site yml.jpg>)

- This file will now be considered as an entry point into the entire infrastructure configuration. 

Other playbooks will be included here as a reference. 

In other words, ***site.yml*** will become a parent to all other playbooks that will be developed. 

Including ***common.yml*** that you created previously.




2. Create a new folder in root of the repository and name it ***static-assignments***. 

The static-assignments folder is where all other children playbooks will be stored. 

This is merely for easy organization of your work. 

It is not an Ansible specific concept, therefore you can choose how you want to organize your work. 

You will see why the folder name has a prefix of static very soon. For now, just follow along.

![static folder](<images/static folder.jpg>)

3. Move common.yml file into the newly created static-assignments folder.

4. Inside site.yml file, import common.yml playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

```

![import playbook](<images/import playbook.jpg>)

The code above uses built in import_playbook Ansible module.

Your folder structure should look like this;

```
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```

5. Run ***ansible-playbook*** command against the *dev* environment

> Since you need to apply some tasks to your *dev* servers and *wireshark* is already installed,

- you can go ahead and create another playbook under ***static-assignments*** and name it ***common-del.yml***. 

In this playbook, configure deletion of ***wireshark*** utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes

```

```
---
- name: update J-Web1, J-Web2,  J-NFS, and J-DB servers
  hosts: J-Web1, J-Web2, J-NFS, J-DB
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update J-LB server
  hosts: J-LB
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
      
```

![common del](<images/common del.jpg>)

- update site.yml with - 

import_playbook: 

```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
``` 

instead of 

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
``` 

![site.yml update](<images/siteyml update.jpg>)

![ansible updated](<images/ansible updated.jpg>)

- Run it against *dev* servers:

```
cd /home/ubuntu/ansible-config-mgt/
```
```
cd /var/lib/jenkins/jobs/Ansible/builds/15/archive/inventory/
```
```
cd /var/lib/jenkins/jobs/Ansible/builds/15/archive/playbooks/
```

![cd into repo](<images/cd into git repo on server.jpg>)

![into folder](<images/run build 1.jpg>)

```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```

```
ansible-playbook -i /var/lib/jenkins/jobs/Ansible/builds/15/archive/inventory/dev.yml /var/lib/jenkins/jobs/Ansible/builds/15/archive/playbooks/site.yml
```
![error](error.jpg)

![error2](images/error2.jpg)


```
eval ssh-agent
```
```
eval `ssh-agent -s`
```
```
ssh-add nfs.pem
```
```
ssh-add -l
```
```
ssh -A ubuntu@3.120.33.54
```
```
cd /var/lib/jenkins/jobs/Ansible/builds/16/archive/
```
```
ansible-playbook -i inventory/dev.yml playbooks/site.yml
```
![playbook success](<images/playbook succes.jpg>)


Make sure that ***wireshark*** is deleted on all the servers by running 

```
wireshark --version
```

![wireshark gone](<images/wireshark gone.jpg>)

Now you have learned how to use import_playbooks module and you have a ready solution to install/delete packages on multiple servers with just one command.

---

# Step 3 - Configure UAT Webservers with a role 'Webserver'

Now we will configure 2 new Web Servers as ***uat***. 

We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly - ***Web1-UAT*** and ***Web2-UAT***.

Note: I have stopped EC2 instances that I am not using at the moment to avoid paying extra. 

For now, We only need 2 new RHEL 8 servers as Web Servers and 1 existing Jenkins-Ansible server up and running.

![RHEL 8 image](<images/RHEL 8 image.jpg>)

![instance failed](<images/instance failed.jpg>)

![RHEL 9](<images/RHEL 9 image.jpg>)

![UAT instances](<images/3 instances for UAT.jpg>)

2. To create a role, you must create a directory called ***roles/***, relative to the playbook file 
or in ***/etc/ansible/*** directory.

There are two ways how you can create this folder structure:

> Use an Ansible utility called ***ansible-galaxy*** inside ***ansible-config-mgt/roles*** directory (you need to create roles directory upfront)

```
cd /var/lib/jenkins/jobs/Ansible/builds/16/archive/
```
```
mkdir roles
```
```
cd roles
```
```
ansible-galaxy init webserver
```

> Create the directory/files structure manually


Note: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.

The entire folder structure should look like below, but if you create it manually - you can skip creating tests, files, and vars or remove them if you used ansible-galaxy

![ansible galaxy](images/galaxy.jpg)


3. Update your inventory ***ansible-config-mgt/inventory/uat.yml*** file with IP addresses of your 2 UAT Web servers.

NOTE: Ensure you are using ssh-agent to ssh into the Jenkins-Ansible instance.

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```
```
[uat-webservers]
172.31.27.10 ansible_ssh_user='ec2-user'
172.31.23.199 ansible_ssh_user='ec2-user'
```

```
[Web1-UAT]
172.31.27.10 ansible_ssh_user='ec2-user'
[Web2-UAT]
172.31.23.199 ansible_ssh_user='ec2-user'
```
```
[uat-webservers]

[Web1-UAT]
172.31.27.10 ansible_ssh_user='ec2-user'

[Web2-UAT]
172.31.23.199 ansible_ssh_user='ec2-user'
```


![uat](images/uat.jpg)

![cat uat](<images/cat uat.jpg>)

4. In ***/etc/ansible/ansible.cfg*** file uncomment ***roles_path*** string 

and provide a full path to your roles directory roles_path    

```
=/home/ubuntu/ansible-config-mgt/roles, 
```
```
[defaults]
host_key_checking=false

inventory=/var/lib/jenkins/jobs/Ansible/builds/16/archive/inventory

roles_path=/var/lib/jenkins/jobs/Ansible/builds/16/archive/roles
```

so Ansible could know where to find configured roles.

![cfg](images/ansible.cfg.jpg)

5. It is time to start adding some logic to the webserver role. 

Go into tasks directory, and within the ***main.yml*** file, start writing configuration tasks to do the following:

-  Install and configure Apache (httpd service)

- Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.

- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.

- Make sure httpd service is started

Your main.yml may consist of following tasks:

```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/ArmstrongLiwox/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
![main.yml](images/main.yml.jpg)

## Step 4 - Reference 'Webserver' role

6. Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. 

This is where you will reference the role.

```
sudo vi uat-webservers.yml
```

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

![uat webservers](<images/uat webservers.jpg>)

![uat servers](<images/uat webservers 1.jpg>)

![uat](<images/uat webservers 2.jpg>)

7. Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.

So, we should have this in site.yml

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

![site.yml](images/site.yml.jpg)

![site](<images/site.ym 1l.jpg>)

## Step 5 - Commit & Test

Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory and see what happens:

```
cd /home/ubuntu/ansible-config-mgt
```
```
cd /var/lib/jenkins/jobs/Ansible/builds/16/archive/
```
```
ansible-playbook -i /inventory/uat.yml playbooks/site.yml
```

> encountered error

![cfg edit](<images/edit cfg.jpg>)


```
[defaults]
host_key_checking=false

inventory=/home/ubuntu/var/lib/jenkins/jobs/Ansible/builds/16/archive/inventory

roles_path=/home/ubuntu/var/lib/jenkins/jobs/Ansible/builds/16/archive/roles
```

![error webserver](images/error3.jpg)

![no match](images/error4.jpg)

![unable to parse](images/error5.jpg)

```
sudo cp -r webserver /etc/ansible
```

![error 6](images/error6.jpg)

> i had a syntax error (there was a / before inventory)
```
ansible-playbook -i inventory/uat.yml playbooks/site.yml
```

![success](<images/error cleared.jpg>)

![successful](images/success.jpg)

> Open port for http everywhere

![open http](<images/open port.jpg>)

- You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:
```
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```

```
http://3.127.108.223/index.php
```
```
http://3.75.193.166/index.php
```
![web 1 success](<images/web 1 success.jpg>)

![web 2 success](<images/web 2 success.jpg>)

My Ansible architecture now looks like this:

![architecture](images/architecture.jpg)

# Success






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








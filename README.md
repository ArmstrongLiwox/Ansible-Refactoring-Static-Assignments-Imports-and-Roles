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





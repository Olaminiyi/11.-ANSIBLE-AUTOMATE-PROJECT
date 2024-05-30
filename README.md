# AUTOMATING PROJECTS 7-10 USING ANSIBLE CONFIGURATION MANAGEMENT

In Projects 7-10, we had to perform lots of manual operations to set up virtual servers, install and configure required software,and deploy our web application.

We will automate the routine tasks with Ansible Configuration Management using declarative language such as `YAML`.

### ANSIBLE CLIENT AS A JUMP SERVER (BASTION HOST)

A **Jump Server(Bastion Host)** is an intermediary server through which access to internal network can be provided. In the current architecture we have been working on, the webservers are inside a secured network which cannot be reached directly from the Internet. To access the webservers using SSH, we need to go through a jump server which provides a better security and reduced attack.

In the diagram below, the `Virtual Private Network (VPC)` is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses

![alt text](images/bastion.png)

We will be performing the following tasks:

- Install and configure Ansible client to act as a **Jump Server/Bastion** Host.
- Create a simple Ansible playbook to automate servers configuration.

### INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![Alt text](images/11.1.png)

In your GitHub account create a new repository and name it `ansible-config-mgt`.

![Alt text](images/11.2.png)

**Install Ansible**
```
sudo apt update
sudo apt install ansible
```
![Alt text](images/11.3.png)

### Jenkins configuration

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
- Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
- Configure Webhook in GitHub and set webhook to trigger ansible build.
- Configure a Post-build job to save all (**) files, like you did it in Project 9.
![Alt text](images/11.4.png)
![Alt text](images/11.5.png)
![Alt text](images/11.6.png)

5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
![Alt text](images/11.7.png)

6. allocate an Elastic IP to your Jenkins-Ansible server 
![Alt text](images/11.8.png)

7. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance
   git clone <ansible-config-mgt repo link>
![Alt text](images/11.9.png)

8. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
![Alt text](images/11.10.png)

9. Checkout the newly created feature branch to your local machine and start building your code and directory structure
    - Create a directory and name it playbooks – it will be used to store all your playbook files.
    - Create a directory and name it inventory – it will be used to keep your hosts organised.
    - Within the playbooks folder, create your first playbook, and name it common.yml
    - Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

![Alt text](images/11.11.png)
![](images/11.12.png)

10. Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:
exit from your ec2 user or ubuntu user(instance) to your local user i.e local directory that contains the pem key file
    - eval `ssh-agent -s`
    - ssh-add <path-to-private-key>(proj7_key.pem)
    - Confirm the key has been added with the command below, you should see the name of your key
    - ssh-add -l
       ![Alt text](images/11.14.png) 
    - then ssh into the ansible-jenkins instance with the public ip address directly from the local directory without using the normal amazon connection
        - ssh -A ubuntu@<public ip address>
        ![Alt text](images/11.13.png)
    - persist the key on the server 
        - ssh-add -l
    - ssh into other instances from the ansible instance with there private keys
        - ssh @ubuntu/ec2-user@<Private ip address>
       ![Alt text](images/11.15.png) 

11. Update your inventory/dev.yml file with this snippet of code:
![Alt text](images/11.16.png)

12. Update your playbooks/common.yml file with following code:
![Alt text](images/11.17.png)

13. use git commands to add, commit and push your branch to GitHub.
    - git status
    - git add <selected files>
    - git commit -m "commit message"
![Alt text](images/11.18.png)

14. Create a Pull request (PR)
![Alt text](images/11.19.png)

15. Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
    - git checkout main
    - git status 
    - git pull .. this will update the master branch with what we have in pr-11 branch
![Alt text](images/11.21.png)

16. Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.
![Alt text](images/11.20.png)

#----------------------------------------------------------------------------------------------------------
17. configure the VScode SSH to connect with the ansible-jenkins server using the public ip in the configuration file
![Alt text](images/11.22.png)

18. create a new folder 'Ubuntu' on the server as a work directory... by clicking the open folder in the vscode menu.. it brings out the name automatically

![Alt text](images/11.23.png)
#----------------------------------------------------------------------------------------------------------

17. ssh to your ansible-jenkins server with ssh ubuntu@<private ip addree>
    - navigate to where you have your project saved on the instance
        - cd /var/lib/jenkins/jobs/ansible/builds/4/archive

18. run the ansible-playbook
    - ansible-playbook -i inventory/dev.yml playbooks/common.yml
    ![Alt text](images/11.24.png)

19. You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version
    - ssh to all the servers from the ansible-jenkins server and check if wireshark has been installed
        - ssh ubuntu/ec2-user@<private ip address>
    ![Alt text](images/11.25.png)
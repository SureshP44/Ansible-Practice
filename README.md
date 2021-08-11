# ansible-firstplaybook
Lab session of the chapter 1 - 3 from Jeff Greeling book


Ansible

Used as an infrastructure as a tool 

USES
    Suppose if we have 10 instances either virtual ones or bare metal machine, if we need to upgrade any simple things like upgrading packages or installing necessary softwares, then it becomes a tedious job. 
    Ansible terms 
        Plays -  play is a simple task like update a package.
        Playbook - its a collections of plays about a single instance or multiple ones.
        Inventory - Trgt machine details.

It has some benefits like
Its has both declarative(yaml) and imperative (cmd line)
Agent-less ( which is we dont need to install ( but its lie the target machine needs python to be pre installed or else we need to install, nowadays all instances comes by preinstalled python) any client like softwares to trgt machines. Unlike chef and puppet will need some agent softwares on trgt machines and they uses Ruby language).
Idempotence ( which is if we have so many configs to do on a trgt machine, but some of them are already satisfied, ansible has a brilliance to skip satisfied ones and it will start from the required ones)
It has more community driven, which is ansible galaxy, where we can find playbooks for all the needs

Ansible installation.    
    It is installed as a package of python
    ‘Pip3 install ansible’

Example 1.    
Inventory file
[ec2instances] -> group name
35.154.73.18 ansible_ssh_private_key_file=/Users/sureshperumal/Downloads/awsudemypractice.pem
52.66.202.44 ansible_ssh_private_key_file=/Users/sureshperumal/Downloads/udemyec2instancepractice2.pem
 Trgt machine IP  To specify the private file

To Run this on terminal
ansible -i inventory ec2instances -m ping -u ec2-user


-i inventory  -  to specify the inventory file name.
-m ping  - module to use
-u ec2-user - user name to login in instances.
--ask-pass - to ask password on runtime if instance is password based authentication
Some ad hoc commands
    Adhoc means to execute come cmds against the instances.
ansible -i inventory ec2instances -a ‘free -h’ -u ec2-user

-a ‘free -h’  -> -a specifies the adhoc cmd and ‘free -h ‘ the command to execute

If we dont specify the module by ‘-m’ it will specify the command module by default.

Some random commands
    Ansible <group_name> -b -i inventory -a ‘<some ad hoc commands>’
    
    In this 
        -b -> it sets the become: true .  By default it will be false. To make the user to sudo we need to change become property ,

    By default when we run a command, it will batch run it into 5 instances at a time, we can make this dynamic by passing fork command in cmd line.
    
    Ansible all -b -i inventory -a ‘date’ fork:1

    Lets assume we have 5 instances , by running the above command, the result of first one prints and then it goes to second one, so on.

    To make commands to run in the background, for a specified period of time, so that we can continue our work synchronously, if it fails we can kill it and return some. To acheive this we have 2 flags ( -B, -P). As a result, it will give some job-id, we can get results after that using async_command.

    Example
ansible -i inventory ec2instances -b -B 3600 -P 0 -a 'yum -y update' -u ec2-user -f 1

Explanation
-i inventory = inventory file name
Ec2instances = group name
-b = makes become: true, so that we can do sudo(super user do)
-B 3600 = to do an asynchronous job for 3600 secs.
-P 0 = if async job fails return 0 
-a ‘yum -y update’ = adhoc command to update yum files.
-u ec2-user = user name for instances
-f 1 = give results for every 1 instances.
    



    To check the status 
     ansible -i inventory 13.233.184.88 -b -m async_status -a "jid=230009875741.5649" -u ec2-user
    
    Explanation:    
        Since the job id is unique for every instances, we have to run for each instances.

13.233.184.88 = ip of target machine
-m async_status = mode of command line
-a “jid = adfasfasfsaf” = to specify the job id.

Writing the first sample Playbook.




Ansible play book can run on either shell script or inbuild modules, in the above example. I was installing the apache and copying some files, and make sure whether the apache server is up and running, we can achieve the same with in build


In the first play ‘ install apache’, takes the yum module, and we are specifying the modules and the parameter state will decide to install(present) or uninstall(absent).


And the second play ‘copy the file’, takes copy module, it requires some field, if we are copying  a single file, we dont need loop(line no 19), copy module takes 4 params, src, dest, owner, group, mode.  


The third play ‘ make sure apache started….’ takes service (denotes package or software) and started param starts the package, and enabled makes running.

And final 4th play copies the local file to instance.


To run the playbook 

Ansible-playbook -i instances -b playbook.yml

If some actions need root user privilege, we can give that directly in play by specifying ‘ become: true’.

If we need to run the playbook on specific group or specific ip, we can do that by using limit command.

Ansible-playbook -i instances -b --limit=<group name / ip address> -b playbook.yml




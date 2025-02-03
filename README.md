# Ansible Short notes:
Ansible consist of 3 main components

- Control Machine
- Inventory
- Playbook

The control machine manages the execution of the Playbook. It can be installed on laptop or on any machine on the internet.

The Inventory file provides a complete list of all the target machines on which various modules are run by doing an ssh connection and install the necessary software’s.

The playbook consists of steps that the control mechanism will perform on the servers defined in the inventory file.

Very important to understand here is that Ansible interacts with all the servers defined in the inventory through the SSH protocol which is a secure method of remote login. Every operation is done and file transfer is encrypted.

So as We would have seen in the previous section Ansible does not use any kind of database for installation and is very easy to install, we will now proceed with the actual usage of Ansible starting with Modules which is the main building block.

----------------------------------------------------------------------------------------------------------------------------------

**Important terms used in Ansible**

Ansible server: The machine where Ansible is installed and from which all tasks and playbooks will be ran

Module: Basically, a module is a command or set of similar commands meant to be executed on the client-side

Task: A task is a section that consists of a single procedure to be completed

Role: A way of organizing tasks and related files to be later called in a playbook

Fact: Information fetched from the client system from the global variables with the gather-facts operation

Inventory: File containing data about the ansible client servers. Defined in later examples as hosts file

Play: Execution of a playbook

Handler: Task which is called only if a notifier is present

Notifier: Section attributed to a task which calls a handler if the output is changed

Tag: Name set to a task which can be used later on to issue just that specific task or group of tasks.

----------------------------------------------------------------------------------------------------------------------------------

**Variables in Ansible** : 

We can give a variable in ‘variable_name: variable_value‘ format. And we can use the variable_name inside double braces anywhere in the playbook. 
eg : 
1. I am declaring a variable hello with value world and is referenced in one of the tasks. The task will output the value world.

        - hosts: all
          vars:
            hello: world
          tasks:
          - name: Ansible Basic Variable Example
            debug:
              msg: "{{ hello }}"

Ansible List (Array) variables  --> We can also have an array or list variables. The following task shows how to declare an array variable in Ansible and how to use the values. The hello contains 9 values, and each one can be accessed using the index numbers(starting from zer0). The following task will output 'South America'
	  
	  - hosts: all
		vars:                               
			hello:
			  - World
			  - Asia
			  - South America
			  - North America
			  - Artic
			  - Antartic
			  - Oceania
			  - Europe
			  - Africa
		tasks:
			- name: Ansible List variable Example
			  debug:
					msg: "{{ hello[2] }}" 

Note : we can also define array as like --> 
vars:
  hello: [Asia, Americas, Artic, Antartic ,Oceania,Europe,Africa]
  
If We need to access all the variables, We can use the with_items structure to loop through all the values. --> 

	- hosts: all
	  vars:
	    hello: [Asia, Americas, Artic, Antartic ,Oceania,Europe,Africa]
	  tasks:
	  - name: Ansible array variables example
	    debug: 
	      msg: "{{ item }}"
	    with_items:
	      - "{{ hello }}"

## Ansible Dictionary Variables (Hash) => 
We can declare dictionaries in Ansible as We would in a python script. And We can access any of the value using the key.  In the following task, I am declaring a variable python which has 3 key-value pairs inside. The following task would print the whole dictionary structure.

        “msg”: {
        “Designer”: “Guido van Rossum”,
        “Developer”: “Python Software Foundation,”
        “OS”: “Cross-platform”
        }	  
        
        *Playbook*
        - hosts: all
          vars:
            python:
              Designer: 'Guido van Rosum'
              Developer: 'Python Software Foundation'
              OS: 'Cross-platform'
          tasks:
          - name: Ansible Dictionary Example
            debug:
              msg: "{{ python }}"
          - name: Ansible Find Example
            debug:
              msg: "{{python.Designer }}"  


**Environment​ Variables in Ansible -->**

**1. Environment Variables in the Local Machine/Ansible Control Machine**
Accessing the environment variablesin the Local Machine/Ansible Control Machine --> Lookup module 
 --> You can access the environment variables available on the local server via the ‘lookup’ plugins, which allow you to access the system data, and ‘env‘ plugin which is written for accessing the environment variables. 	  
 
 Suppose we need to print the value of the ‘HOME’ environment variable. We can execute the following task.
 
         - hosts: all
          tasks:
            - name: Printing the environment​ variable in Ansible
              debug:
                msg: "{{ lookup('env','HOME') }}"

output
------
"msg": "/Users/mdtutorials2"

**2. Environment Variables in the Remote Servers/ Target Machines**
Since the lookup plugins work only on the local machines, we need some other way to access the variables on the remote servers.
--> The environment variables of the remote servers can be accessed via ‘facts‘. There is a sublist called ‘ansible_env’ which has all the env variables inside it.
Note that, the fact-gathering needs to be ‘ON’ if you want to access this. By default, it is set to true. Or you can set the ‘gather_facts’ to true explicitly.

Suppose I want to accès the ‘HOME’ variable on the remote servers and print it, I can do the following.

        - hosts: all
          gather_facts: true
          tasks:
            - name: Remote server ansible variables
              debug:
                msg: "{{ ansible_env.HOME }}"

output
------
"msg": "/Users/mdtutorials3"

----------------------------------------------------------------------------------------------------------------------------------

**Ansible loop :**
install a lot of Linux packages can be written like the below example.
instead of writing 3 separate task we have consolidated them into a single task.
In each iteration, the value of with_items block will be inserted in place of {{ item }}. 

        ---
        hosts: all 
        become : true 
        tasks:
         - name: Ansible Loop example
          apt:
            name: "{{ item }}"
            state: present
          with_items:
             - python3
             - ca-certificates
             - git
  
----------------------------------------------------------------------------------------------------------------------------------

## Ansible Modules
Modules are the main building blocks of Ansible and are basically reusable scripts that are used by Ansible playbooks. 
Ansible comes with a number of reusable modules. These include functionality for controlling services, software package installation, working with files and directories etc.

Ansible modules take arguments in key-value pairs that look similar to key=value --> perform a job on the remote server, and return information about the job as JSON.
The key-value pairs allow the module to know what to do when requested. They can be hardcoded values, or in playbooks they can use variables

Syntax : 
$ansible hostORgroup -m module_name -a "arguments" -u username --become

Module examples : 
1. Setup module : This module connects to the configured node, gathers data about the system, and then returns those values.

$ ansible webservers –m setup  # To get information about the network or hardware or OS version or memory related information the setup module will help to gather the same about the target machines.

2.Command Module --> The command module simply executes a specific command on the target machine and gives the output.

	$ ansible webservers –m command - an ‘uptime’
	$ ansible webservers –m command –a ‘hostname’

3) Shell Module --> To execute any command in the shell of Wer choice We can use the Shell module. The shell module commands are run in /bin/sh shell and We can make use of the operators like ‘>’ or ‘|’ (pipe symbol or even environment variables.

Difference between Shell & Command ?
--> if we dont want to use the operators like -a, -an etc then We could use the command module.

4) User Module --> Using this module one can create or delete users.
to add :

        $ ansible webservers -m user -a 'name=user1 password=user1' --become
   
to delete : 

        $ ansible webservers -m user -a 'name=user1 state=absent' --become

Note:
Options:
        become – Privilege to the superuser to run the command
        state=absent to delete the user

5) File Module --> This module is used to create files, directories, set, or change file permissions and ownership etc

Example 1: Create a file

        $ ansible webservers -m file -a 'dest=/home/ansible/gaurang.txt state=touch mode=600 owner=ansible group=ansible' 

6. Set_fact --> The set_fact module allows us to build our own facts on the machine inside an Ansible play. 
These facts can then be used inside templates or as variables in the playbook. 

7. pause --> The pause module stops the execution of a playbook for a certain period of time. We can configure it to wait for a particular period, or we can make it prompt for the user to continue.
Generally, the pause module is used when we want the user to provide confirmation to continue, or if manual intervention is required at a particular point. 
For example, if we have just deployed a new version of a web application to a server, and we need to have the user check manually to make sure it looks okay before we configure them to receive production traffic, 
we can put a pause there

8. Cloud Infrastructure modules -->  Infrastructure modules allow us to not only manage the setup of our machines, but also the creation of those machines themselves. Apart from this, we can also automate much of the infrastructure surrounding them. 

9. Delete a file
    
        $ ansible webservers -m file -a "dest=/home/ansible/gaurang.txt state=absent"
10. Create a directory --> To create a directory using the file module, We need to set two parameters.

Path(alias – name, dest) – This is the absolute path of the directory to be created.
State – We should enter the value as ‘directory.’ By default, the value is ‘file’.

        $ ansible webservers -m file -a "dest=/home/ansible/vndir state=directory mode=755"
        
11. Managing Services Module --> To manage services with ansible, we use a module ‘service’.

Starting a service -->

        $ ansible webservers -m service -a “name=httpd state=started” --become
        
Stopping a service --> 
        
        $ ansible webservers -m service -a “name=httpd state=stopped” --become 
        
Restart service -->

        $ ansible webservers -m service -a “name=httpd state=restarted --become


----------------------------------------------------------------------------------------------------------------------------------
**Ansible Playbooks -->** 

We have seen how to run single tasks or one time tasks using Modules, but what if We need to execute multiple tasks? Playbooks help to run them in a scripted way.
Playbooks define variables, configurations, deployment steps, assign roles, perform multiple tasks. For E.g. COPY / DELETE Files and Folders, install packages, start services. 
So primarily playbooks are defined to orchestrate the steps to multiple machines or servers and get them all to a certain desired state.

Playbook is written in YAML format with a .yml file extension. One needs to be very careful with the format and alignment which makes it very sensitive.

It contains the following sections:

Every playbook starts with 3 hyphens '---' and ends with 3 dots '...'
Host section – Defines the target machines on which the playbook should run. This is based on the Ansible inventory file.
Variable section – This is optional and can declare all the variables needed in the playbook. We will look at some examples as well.
Tasks section – This section lists out all the tasks that should be executed on the target machine. It specifies the use of Modules. Every task has a name which is a small description of what the task will do and will be listed while the playbook is run.

How To run any playbook -->  

        $ ansible-playbook <playbook.yml>

How To check the playbook for syntax errors -->  

        $ ansible-playbook <playbook.yml> --syntax-check

To view hosts list --> 

        $ ansible-playbook <playbook.yml> --list-hosts

        
----------------------------------------------------------------------------------------------------------------------------------

**Ansible Roles** : 
Its another level of abstraction used to organise Playbooks.
Ansible roles allow us to configure groups of nodes at the same time, without repeating ourselves. Roles also provide a way to create modular code, which then can then be shared and reused in playbooks.
it is a group of variables, tasks, files, and handlers that are stored in a standardized file structure.
Roles are not playbooks. Roles are small functionality which can be independently used but have to be used within playbooks

ansible-galaxy [delete|import|info|init|install|list|login|remove|search|setup] [--help] [options] ... 

	$ ansible-galaxy init garryrole 
        
        $ tree garryrole/ 
        garryrole/ 
        ├── defaults 
        │   └── main.yml 
        ├── files ├── handlers 
        │   └── main.yml 
        ├── meta 
        │   └── main.yml 
        ├── README.md ├── tasks 
        │   └── main.yml 
        ├── templates ├── tests │   ├── inventory 
        │   └── test.yml 
        └── vars 
            └── main.yml 

----------------------------------------------------------------------------------------------------------------------------------

**Handlers** --> Its like regular task but it will run only when task contains 'notify' directive 
Example:

        ---
        - hosts: all
          tasks:
           - name: install tomcat
             yum: name=httpd state=present
             notify:
              - start tomcat
          handlers:
           - name: start tomcat
             service: name=httpd state=started
 
----------------------------------------------------------------------------------------------------------------------------------

**Ansible Vaults** --> are files that are encrypted and can be decrypted by Ansible transparently. We can use them for includes, variable files, tasks lists in roles and any other YAML formatted file that Ansible uses. We can also use it with both JSON and YAML files included with the -e command-line argument to ansible-playbook. Vault files are managed with the ansible-vault command and can be used as if they were not encrypted at all.

The ansible-vault command has several modes, which are given as the first argument. This table describes the modes:

Mode                     Action
Create            This starts Wer default editor to create a new encrypted file
Encrypt           This encrypts an existing file, turning it into a vault
Edit              This edits a vault allowing We to change the content
Rekey             This changes the password that is used to encrypt the vault
Decrypt           This decrypts the vault turning it back into a regular file

----------------------------------------------------------------------------------------------------------------------------------

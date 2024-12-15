# Introduction to Ansible: Simplifying IT Automation

# What is Ansible?

> Ansible is an open-source tool that helps automate IT tasks. With Ansible, we can manage configuration settings, deploy applications, and automate repetitive tasks. It allows us to manage infrastructure, deploy software, and perform tasks across multiple systems easily, making it a simple and efficient solution for automation.
> 

# Why Ansible?

> When a developer writes code on their personal laptop and confirms the application works as expected, the challenge arises in replicating that same working environment across other stages like testing, staging, production, and even the customer's setup. This is where configuration management tools like Ansible Play.
> 
> 
> Instead of manually configuring each system—installing dependencies, setting up environments, and applying configurations—Ansible automates the entire process. It ensures that all systems, regardless of their location, are configured consistently and correctly, saving time and reducing errors.
> 

# How to install Ansible:

Follow these steps to install Ansible on your system:

Step 1 : Update the package list

```jsx
sudo apt update  
```

Step 2 : Install Ansible

```jsx
sudo apt install ansible  
```

Step 3 : Verify the installation

```jsx
ansible --version  
```

# Prerequisite for Ansible :

Step 1 :  Create two EC2 instances on any cloud provider (e.g., AWS, Azure, or GCP). In this example, we use AWS. Name the first instance for eg  `ansible-server` and the second instance `target-server`  the instances name would be your choice

Step 2 :  Log in to both `EC2 instances`. You can choose your preferred method; here, i log in using the `AWS CLI` (Command Line Interface).

Step 3 :  After successfully logging in to both instances, create a folder named `ansible` on the `ansible-server` and navigate into it using: 

```jsx
mkdir ansible  
cd ansible  
```

Step 4:  Generate an SSH key pair on the `ansible-server` by running: 

```jsx
ssh-keygen    
```

 After that, press **Enter** to accept the default file location and key settings.

Step 5 :  After completing the process, the terminal will display the path to the private key and public key typically : `/home/ubuntu/.ssh/id_rsa`.

Step 6 :  Verify the generated key pair by listing the files in the `.ssh` directory: 

```jsx
ls /home/ubuntu/.ssh/  
```

You should see files like `authorized_keys`, `id_rsa`, `id_rsa.pub`, and `known_hosts`. 

Step 7 :  Display the public key by running: 

```jsx
cat /home/ubuntu/.ssh/id_rsa.pub  // Copy the public key displayed in the terminal.
```

Step 8 :  Log in to the `target-server` and generate an SSH key pair there as well using:

```jsx
ssh-keygen  
```

Step 9 :  Verify the `.ssh` directory on the `target-server` by running: 

```jsx
ls /home/ubuntu/.ssh/  
```

Step 10 :  Open the `authorized_keys` file on the `target-server` and paste the public key from the `ansible-server` into it: 

```jsx
vim /home/ubuntu/.ssh/authorized_keys  // Save and exit the file.
```

Step 11 :  Now, return to the `ansible-server` and test the connection to the `target-server` by running:

```jsx
ssh [target-server-private-ip-address]  // example: ssh 172.68.37.100  
```

After running all this commands we can successfully do a password less authentication from first EC2 Instance (ansible-server) to Second EC2 Instance (target-server)

# Verifying Password-less Authentication by Creating a File

 To confirm that password-less authentication is working correctly, we will create a file on the `ansible-server` and ensure it appears on the `target-server`. This demonstrates seamless communication and file execution between the two servers using Ansible.

### Steps to Create and Verify a File Using Ansible :

Step 1 :  Log out of the `target-server` by running below command in `ansible-server`:

```jsx
logout   
```

  This returns you to the ansible-server. You should now see the ansible-server IP address in the terminal prompt. Initially, it displayed the target-server IP because we had set up passwordless authentication between the two servers.

Step 2 :  Create a file called `inventory` on the `ansible-server`:

```jsx
touch inventory  
```

Step 3 :  Open the `inventory` file and add the private IP address of the `target-server`:

```jsx
vim inventory  
```

Paste the private IP address of the `target-server` in the file, save, and close it.

Step 4 :  Verify the contents of the `inventory` file using:

```jsx
 cat inventory  
//If the setup is correct, you should see output similar to: 172.31.16.100 | changed | rc = 0  
```

Step 5 :  Run the following Ansible ad hoc command to create a file named `devops_file` on the `target-server`  file name would be your choice :

```jsx
ansible -i inventory all -m "shell" -a "touch devops_file"  
```

- Here’s a breakdown of the command:
    - **`ansible`**: The main Ansible CLI command used to execute tasks on remote servers.
    - **`-i inventory`**: Specifies the inventory file containing the target server's IP address.
    - **`all`**: Targets all servers listed in the inventory file (in this case, only the `target-server`).
    - **`-m "shell"`**: Specifies the module to be used. The `shell` module allows us to run shell commands on the target server.
    - **`-a "touch devops_file"`**: Provides arguments to the module. Here, the `touch devops_file` command creates an empty file named `devops_file` on the `target-server`.

Step 6 :  Switch to the `target-server` and run the following command:

```jsx
ls  
// You should see devops_file listed in the directory, confirming that the file was successfully created on the target-server.
```

This process verifies that passwordless authentication and Ansible are set up correctly, enabling seamless task execution across servers.

# Ansible Playbooks :

### What is a Playbook in Ansible?

- A playbook in Ansible is essentially a YAML file that contains instructions for automating tasks
- Similar to how we refer to files in other programming environments (e.g., shell scripts, Python files), in Ansible, we call them playbooks
- Ansible tasks can be performed using either **ad-hoc commands or playbooks**:
    - Use **ad-hoc commands** for smaller, one-time tasks
    - Use **playbooks** for larger, more complex, or repetitive tasks involving multiple steps

### Why Use Playbooks?

- Playbooks allow you to define and automate multiple tasks in a structured and repeatable manner
- Playbooks are written in **YAML** (Yet Another Markup Language), which is easy to read and write

### Example: Install and Start NGINX Using a Playbook :

Step 1 :  Create a Playbook File

Run the following command to create a file named `first-playbook.yml`:

```jsx
vim first-playbook.yml  
```

Step 2 :  Write the Playbook Content

```jsx
---         # Indicates the start of a YAML file  
- name: Install and Start NGINX    # Description of the playbook  
  hosts: all    # Specifies the target hosts (from inventory)  
  become: true    # Allows the playbook to run with sudo privileges  

  tasks:  
    - name: Install NGINX    # First task: Install NGINX  
      apt:  
        name: nginx  
        state: present  

    - name: Start NGINX      # Second task: Start NGINX  
      service:  
        name: nginx  
        state: started  

```

Step 3 :  Run the Playbook

Execute the playbook using the following command:

```jsx
ansible-playbook -i inventory first-playbook.yml  
```

Step 4 : Verify NGINX Installation 

Log in to the `target-server` and check the status of NGINX using:

```jsx
sudo systemctl status nginx  

//If everything is set up correctly, this command will confirm that NGINX is installed and running.
```

### Writing Multiple Playbooks in a Single File :

**Scenario:** Install and Start Both NGINX and Apache

Step 1 : Create a New Playbook File

Run below command

```jsx
vim multiple-playbooks.yml  
```

Step 2 : Write the Playbook Content

```jsx
# First Playbook: Install and Start NGINX 
---      
- name: Install and Start NGINX  
  hosts: all  
  become: true  

  tasks:  
    - name: Install NGINX  
      apt:  
        name: nginx  
        state: present  

    - name: Start NGINX  
      service:  
        name: nginx  
        state: started  

# Second Playbook: Install and Start Apache 
---         
- name: Install and Start Apache  
  hosts: all  
  become: true  

  tasks:  
    - name: Install Apache  
      yum:  
        name: httpd  
        state: present  

    - name: Start Apache  
      service:  
        name: httpd  
        state: started  

```

Step 3 : Run the Playbook

```jsx
ansible-playbook -i inventory multiple-playbooks.yml  
```

### Key Points :

- **Playbooks vs Ad-Hoc Commands**: Playbooks are better for complex, multi-step tasks, while ad-hoc commands are great for quick one-off tasks
- **YAML Structure**: Ensure the playbook follows YAML syntax, with proper indentation and structure
- **Multiple Playbooks**: You can write multiple playbooks in a single file using `--` to separate them

# Ansible Roles: Simplifying Playbooks

### What Are Ansible Roles?

- Ansible Roles help organize and simplify complex playbooks by breaking them into smaller, reusable components
- They provide a modular structure, making it easier to manage configuration, variables, templates, and tasks in a well-organized directory format
- Roles are especially useful when writing complicated playbooks

### Why Use Ansible Roles?

- Simplify complex playbooks by dividing them into logical sections
- Promote **reusability** and **modularity** in your automation scripts
- Improve **maintainability** by organizing files and tasks in a structured way

### What Is Ansible Galaxy?

- **Ansible Galaxy** is a platform and command-line tool for sharing and reusing Ansible Roles.
- It provides access to a large repository of pre-built roles contributed by the Ansible community.
- You can use Ansible Galaxy to download roles and integrate them into your projects, saving time and effort.

# How to Create and Use Ansible Roles

### Example: Installing Kubernetes Using Ansible Roles

Step 1: Create a Role

Run the following command to initialize a new role:

```jsx
ansible-galaxy role init kubernetes  

// If successful, you'll see the message: Role 'kubernetes' was created successfully.  

```

This command creates a directory structure for the `kubernetes` role, including subdirectories for tasks, handlers, templates, and more.

Step 2: Push the Role to GitHub

To share the role or keep it version-controlled, push it to a GitHub repository:

Navigate to the role's directory:

```jsx
cd /path/to/your/project  
```

Initialize a Git repository:

```jsx
git init  
```

Stage and commit the files:

```jsx
git add .  
git commit -m "Initial commit of Ansible role"  
```

Add your GitHub repository as a remote:

```jsx
git remote add origin https://github.com/username/ansible-roles.git  
```

Push the role to github

```jsx
git branch -M main  
git push -u origin main  

// Verify the files on GitHub to ensure everything is uploaded.
```

### Benefits of Using Roles :

- **Clean Directory Structure**: Roles separate tasks, variables, templates, and files into different directories for better organization.
- **Easy Sharing**: Roles can be shared or reused across multiple projects
- **Collaboration-Friendly**: Version control with platforms like GitHub makes it easy for teams to collaborate on roles
- **Ansible Galaxy Integration**: Pre-built roles from Ansible Galaxy allow you to use community-created solutions or share your roles with others

By adopting Ansible Roles, you can make your automation workflows more efficient, maintainable, and scalable 

# Conclusion :

Ansible is a versatile tool for automating IT tasks, managing infrastructure, and deploying applications efficiently. With features like Playbooks, Roles, and Ad-Hoc Commands, it simplifies complex workflows and ensures consistency across environments. Ansible’s modularity and integration with tools like Ansible Galaxy make it ideal for both small and large-scale projects.

# Resources :

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/ui/)
- [Ansible Github](https://github.com/ansible/ansible)








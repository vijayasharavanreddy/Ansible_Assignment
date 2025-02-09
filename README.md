# Ansible Web Server Deployment

## Overview
This project automates the deployment and undeployment of an Apache web server on multiple EC2 instances using Ansible. The playbook installs, configures, and starts the web server on port 8080, displaying a custom message. It also provides an option to undeploy by stopping and uninstalling Apache.

## Prerequisites
- Ansible installed on your local machine
- SSH access to the EC2 instances
- A valid SSH key
- Inventory file listing the EC2 instances

## Setup

### 1. Clone the Repository
```sh
git clone <repository-url>
cd <repository-name>
```

### 2. Configure Inventory
Create an inventory file (e.g., `inventory.ini`) with the following format:
```
[webservers]
vm1 ansible_host=<VM1_IP> ansible_user=ubuntu ansible_ssh_private_key_file=<PATH_TO_KEY>
vm2 ansible_host=<VM2_IP> ansible_user=ubuntu ansible_ssh_private_key_file=<PATH_TO_KEY>
```

### 3. Create the Ansible Playbook
Create a `webserver.yml` file with the following contents:
```yaml
---
- name: Manage webserver
  hosts: webservers
  become: true
  vars:
    deploy: true  # Set to false for undeployment
  tasks:
    - name: Install Apache (deploy only)
      apt:
        name: apache2
        state: present
        update_cache: true
      when: deploy
    
    - name: Configure Apache to listen on port 8080 (deploy only)
      lineinfile:
        path: /etc/apache2/ports.conf
        regexp: '^Listen 80'
        line: 'Listen 8080'
        state: present
      when: deploy
    
    - name: Update VirtualHost to port 8080 (deploy only)
      replace:
        path: /etc/apache2/sites-available/000-default.conf
        regexp: '(<VirtualHost \*:)(80)(>)'
        replace: '\1 8080\3'
      when: deploy
    
    - name: Restart Apache (deploy only)
      service:
        name: apache2
        state: restarted
      when: deploy
    
    - name: Create custom index.html (deploy only)
      copy:
        dest: /var/www/html/index.html
        content: |
          Hello World from {{ inventory_hostname }}
      when: deploy
    
    - name: Stop Apache (undeploy only)
      service:
        name: apache2
        state: stopped
      when: not deploy
    
    - name: Remove index.html (undeploy only)
      file:
        path: /var/www/html/index.html
        state: absent
      when: not deploy
    
    - name: Uninstall Apache (undeploy only)
      apt:
        name: apache2
        state: absent
      when: not deploy
```

### 4. Running the Playbook

#### Deploy Web Server
```sh
ansible-playbook -i inventory.ini webserver.yml --extra-vars "deploy=true"
```

#### Undeploy Web Server
```sh
ansible-playbook -i inventory.ini webserver.yml --extra-vars "deploy=false"
```

### 5. Verify Deployment
- Open a browser and navigate to `http://<VM1_IP>:8080` or `http://<VM2_IP>:8080`
- Run:
```sh
curl -I http://<VM1_IP>:8080
```

## License
This project is open-source and free to use.


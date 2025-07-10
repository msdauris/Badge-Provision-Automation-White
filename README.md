# Badge-Provision-Automation-White
Badge Provision Automation White

# Provision Automation Tools Study Notes & Practical Lab

## 1. Overview of Provision Automation Tools

### What is Provision Automation?
Provision automation tools automate the process of configuring and managing IT infrastructure, ensuring consistent, repeatable deployments across environments.

### Key Benefits
- **Consistency**: Eliminates configuration drift
- **Speed**: Rapid deployment and scaling
- **Reliability**: Reduces human error
- **Documentation**: Infrastructure becomes self-documenting
- **Version Control**: Track changes over time

---

## 2. Comparison: Ansible vs. Puppet vs. Chef

### Ansible
```yaml
# Example Ansible playbook
- name: Install and configure Apache
  hosts: webservers
  become: yes
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

**Characteristics:**
- **Agent**: Agentless (uses SSH)
- **Language**: YAML (declarative)
- **Architecture**: Push-based
- **Learning Curve**: Easy to learn
- **Execution**: Sequential by default
- **Inventory**: Simple text files or dynamic
- **Best For**: Simple automation, cloud provisioning, orchestration

### Puppet
```puppet
# Example Puppet manifest
class apache {
  package { 'httpd':
    ensure => installed,
  }
  
  service { 'httpd':
    ensure => running,
    enable => true,
    require => Package['httpd'],
  }
}
```

**Characteristics:**
- **Agent**: Agent-based (puppet agent)
- **Language**: DSL (Domain Specific Language)
- **Architecture**: Pull-based
- **Learning Curve**: Steeper learning curve
- **Execution**: Dependency-based
- **Inventory**: Puppet Master manages nodes
- **Best For**: Large-scale enterprise environments, compliance

### Chef
```ruby
# Example Chef recipe
package 'httpd' do
  action :install
end

service 'httpd' do
  action [:enable, :start]
end
```

**Characteristics:**
- **Agent**: Agent-based (chef-client)
- **Language**: Ruby DSL
- **Architecture**: Pull-based
- **Learning Curve**: Requires Ruby knowledge
- **Execution**: Sequential with dependencies
- **Inventory**: Chef Server manages nodes
- **Best For**: DevOps teams with Ruby expertise

### Quick Comparison Table

| Feature | Ansible | Puppet | Chef |
|---------|---------|---------|------|
| **Agent** | Agentless | Agent-based | Agent-based |
| **Language** | YAML | DSL | Ruby DSL |
| **Architecture** | Push | Pull | Pull |
| **Learning Curve** | Easy | Medium | Hard |
| **Configuration** | Playbooks | Manifests | Recipes/Cookbooks |
| **Execution Order** | Sequential | Dependency-based | Sequential |
| **Best For** | Quick automation | Enterprise compliance | Ruby-savvy teams |

---

## 3. Infrastructure as Code (IaC) Concepts

### Infrastructure as Code (IaC)
**Definition**: Managing and provisioning computing infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

**Key Principles:**
- **Declarative**: Define the desired state, not the steps
- **Version Controlled**: Track changes using Git
- **Immutable**: Replace rather than modify
- **Idempotent**: Same result regardless of execution count

### Snowflake Servers
**Definition**: Servers that are manually configured and become unique over time, making them difficult to reproduce or replace.

**Problems with Snowflake Servers:**
- Configuration drift
- Difficult to troubleshoot
- Cannot be easily reproduced
- Manual processes are error-prone
- Scaling becomes complex

**Solution**: Use IaC to create reproducible, consistent server configurations.

### Thin Client/Provisioning
**Definition**: Approach where servers are provisioned with minimal base configuration, and applications/services are deployed separately.

**Benefits:**
- Faster provisioning
- Easier updates and rollbacks
- Better separation of concerns
- More flexible deployment strategies

**Example Architecture:**
```
Base Image (AMI/Template) → Thin Provisioning → Application Deployment
```

### Configuration Management vs. Orchestration
- **Configuration Management**: Ansible, Puppet, Chef (configure servers)
- **Orchestration**: Terraform, CloudFormation (provision infrastructure)
- **Combined Approach**: Use both for complete IaC solution

---

## 4. Practical Lab: Ansible Playbook Demo

### Lab Objective
Create an Ansible playbook that:
1. Displays system information (date, time, OS)
2. Installs and configures Apache
3. Creates a custom webpage
4. Demonstrates infrastructure as code principles

### Prerequisites
- Linux VM or EC2 instance (control node)
- SSH access to target servers
- Basic understanding of YAML

---

## 5. Step-by-Step Ansible Setup

### Step 1: Install Ansible on Control Node
```bash
# Amazon Linux/RHEL/CentOS
sudo yum update -y
sudo yum install -y epel-release
sudo yum install -y ansible

# Ubuntu/Debian
sudo apt update
sudo apt install -y ansible

# Verify installation
ansible --version
```

### Step 2: Create Project Directory Structure
```bash
# Create project directory
mkdir -p ~/ansible-lab
cd ~/ansible-lab

# Create directory structure
mkdir -p {playbooks,inventory,roles,group_vars,host_vars}

# Create files
touch inventory/hosts
touch playbooks/site.yml
touch ansible.cfg
```

### Step 3: Configure Ansible
```bash
# Create ansible.cfg
cat > ansible.cfg << 'EOF'
[defaults]
inventory = inventory/hosts
remote_user = ec2-user
private_key_file = ~/.ssh/your-key.pem
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = memory

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = ~/.ansible/cp/%%h-%%p-%%r
EOF
```

### Step 4: Create Inventory File
```bash
# Create inventory/hosts
cat > inventory/hosts << 'EOF'
[webservers]
web1 ansible_host=10.0.11.157 ansible_user=ec2-user
web2 ansible_host=10.0.12.254 ansible_user=ec2-user

[webservers:vars]
ansible_ssh_private_key_file=~/.ssh/your-key.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[all:vars]
ansible_python_interpreter=/usr/bin/python3
EOF
```

### Step 5: Test Connection
```bash
# Test connection to all hosts
ansible all -m ping

# Expected output:
# web1 | SUCCESS => {
#     "ansible_facts": {
#         "discovered_interpreter_python": "/usr/bin/python3"
#     },
#     "changed": false,
#     "ping": "pong"
# }
```

### Step 6: Create Main Playbook
```yaml
---
- name: System Information and Apache Setup
  hosts: webservers
  become: yes
  gather_facts: yes
  
  vars:
    apache_package: httpd
    apache_service: httpd
    web_root: /var/www/html
    
  tasks:
    - name: Display system information
      debug:
        msg: |
          System Information:
          - Hostname: {{ ansible_hostname }}
          - OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          - Architecture: {{ ansible_architecture }}
          - Date/Time: {{ ansible_date_time.iso8601 }}
          - Uptime: {{ ansible_uptime_seconds | int // 3600 }} hours
          - Memory: {{ ansible_memtotal_mb }}MB
          - CPU Cores: {{ ansible_processor_cores }}
      
    - name: Install Apache HTTP Server
      yum:
        name: "{{ apache_package }}"
        state: present
        
    - name: Start and enable Apache service
      systemd:
        name: "{{ apache_service }}"
        state: started
        enabled: yes
        
    - name: Create system info webpage
      template:
        src: system_info.html.j2
        dest: "{{ web_root }}/system_info.html"
        owner: apache
        group: apache
        mode: '0644'
      notify: restart apache
      
    - name: Create custom index page
      template:
        src: index.html.j2
        dest: "{{ web_root }}/index.html"
        owner: apache
        group: apache
        mode: '0644'
      notify: restart apache
      
    - name: Open firewall for HTTP
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes
      ignore_errors: yes
      
    - name: Verify Apache is responding
      uri:
        url: "http://{{ ansible_default_ipv4.address }}"
        method: GET
        status_code: 200
      delegate_to: localhost
      become: no
      
  handlers:
    - name: restart apache
      systemd:
        name: "{{ apache_service }}"
        state: restarted
```

### Step 7: Create Templates
```bash
# Create templates directory
mkdir -p templates

# Create index.html template
cat > /home/ec2-user/ansible-lab/playbooks/templates/index.html.j2 << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to {{ ansible_hostname }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background-color: #f0f0f0; }
        .container { background-color: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1 { color: #2c3e50; }
        .links { margin-top: 20px; }
        a { color: #3498db; text-decoration: none; margin-right: 20px; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to {{ ansible_hostname }}</h1>
        <p>This server is managed by Ansible!</p>
        <p><strong>Server:</strong> {{ ansible_hostname }}</p>
        <p><strong>IP Address:</strong> {{ ansible_default_ipv4.address }}</p>
        <div class="links">
            <a href="/system_info.html">View System Information</a>
        </div>
    </div>
</body>
</html>
EOF
```

```bash
# Create detailed system info template
cat > templates/system_info.html.j2 << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>System Information - {{ ansible_hostname }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .section { margin: 20px 0; padding: 15px; background-color: #f9f9f9; border-radius: 5px; }
        table { width: 100%; border-collapse: collapse; }
        th, td { text-align: left; padding: 8px; border-bottom: 1px solid #ddd; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <h1>Detailed System Information</h1>
    
    <div class="section">
        <h2>Basic Information</h2>
        <table>
            <tr><th>Property</th><th>Value</th></tr>
            <tr><td>Hostname</td><td>{{ ansible_hostname }}</td></tr>
            <tr><td>FQDN</td><td>{{ ansible_fqdn }}</td></tr>
            <tr><td>Operating System</td><td>{{ ansible_distribution }} {{ ansible_distribution_version }}</td></tr>
            <tr><td>Kernel</td><td>{{ ansible_kernel }}</td></tr>
            <tr><td>Architecture</td><td>{{ ansible_architecture }}</td></tr>
            <tr><td>Current Date/Time</td><td>{{ ansible_date_time.iso8601 }}</td></tr>
            <tr><td>Uptime</td><td>{{ ansible_uptime_seconds | int // 3600 }} hours, {{ (ansible_uptime_seconds | int % 3600) // 60 }} minutes</td></tr>
        </table>
    </div>
    
    <div class="section">
        <h2>Hardware Information</h2>
        <table>
            <tr><th>Property</th><th>Value</th></tr>
            <tr><td>Processor</td><td>{{ ansible_processor[1] }} ({{ ansible_processor_cores }} cores)</td></tr>
            <tr><td>Memory Total</td><td>{{ ansible_memtotal_mb }}MB</td></tr>
            <tr><td>Memory Free</td><td>{{ ansible_memfree_mb }}MB</td></tr>
            <tr><td>Swap Total</td><td>{{ ansible_swaptotal_mb }}MB</td></tr>
        </table>
    </div>
    
    <div class="section">
        <h2>Network Information</h2>
        <table>
            <tr><th>Property</th><th>Value</th></tr>
            <tr><td>Default IPv4</td><td>{{ ansible_default_ipv4.address }}</td></tr>
            <tr><td>Default Gateway</td><td>{{ ansible_default_ipv4.gateway }}</td></tr>
            <tr><td>DNS Servers</td><td>{{ ansible_dns.nameservers | join(', ') }}</td></tr>
        </table>
    </div>
    
    <div class="section">
        <h2>Disk Information</h2>
        <table>
            <tr><th>Mount Point</th><th>Size</th><th>Used</th><th>Available</th><th>Use%</th></tr>
            {% for mount in ansible_mounts %}
            <tr>
                <td>{{ mount.mount }}</td>
                <td>{{ (mount.size_total / 1024 / 1024 / 1024) | round(1) }}GB</td>
                <td>{{ ((mount.size_total - mount.size_available) / 1024 / 1024 / 1024) | round(1) }}GB</td>
                <td>{{ (mount.size_available / 1024 / 1024 / 1024) | round(1) }}GB</td>
                <td>{{ ((mount.size_total - mount.size_available) / mount.size_total * 100) | round(1) }}%</td>
            </tr>
            {% endfor %}
        </table>
    </div>
    
    <p><a href="/">← Back to Main Page</a></p>
</body>
</html>
EOF
```

### Step 8: Run the Playbook
```bash
# Run the playbook
ansible-playbook playbooks/site.yml

# Run with verbose output
ansible-playbook playbooks/site.yml -v

# Run with check mode (dry run)
ansible-playbook playbooks/site.yml --check

# Run on specific hosts
ansible-playbook playbooks/site.yml --limit web1
```

### Step 9: Verify Results
```bash
# Check if Apache is running on targets
ansible webservers -m command -a "systemctl status httpd"

# Check web content
ansible webservers -m uri -a "url=http://{{ ansible_default_ipv4.address }} return_content=yes"

# Manual verification
curl http://10.0.11.157
curl http://10.0.12.254
```

---

## 6. Advanced Ansible Concepts

### Roles
```bash
# Create role structure
ansible-galaxy init roles/webserver

# Use role in playbook
- name: Deploy web servers
  hosts: webservers
  roles:
    - webserver
```

### Vault (Secure Secrets)
```bash
# Create encrypted variable file
ansible-vault create group_vars/all/vault.yml

# Edit vault file
ansible-vault edit group_vars/all/vault.yml

# Run playbook with vault
ansible-playbook playbooks/site.yml --ask-vault-pass
```

### Dynamic Inventory
```bash
# AWS EC2 dynamic inventory
ansible-playbook playbooks/site.yml -i aws_ec2.yml

# Custom dynamic inventory script
ansible-playbook playbooks/site.yml -i scripts/custom_inventory.py
```

---

## 7. Evidence Collection for Badge

### JIRA Time Tracking
```
Task: "Set up Ansible automation for web server provisioning"
Time: 4-6 hours
Description: 
- Installed and configured Ansible
- Created playbooks for Apache installation
- Implemented system information gathering
- Tested across multiple environments
```

### BitBucket Commits
```bash
# Initialize git repository
git init
git add .
git commit -m "Initial Ansible playbook for web server provisioning"

# Commit structure should show:
git log --oneline
# abc123 Add system information display functionality
# def456 Implement Apache installation and configuration
# ghi789 Create HTML templates for web content
# jkl012 Add firewall and security configurations
# mno345 Initial Ansible project setup
```

### Documentation
Create a README.md file documenting:
- Project purpose and scope
- Prerequisites and setup instructions
- Playbook structure and components
- How to run and test
- Troubleshooting common issues

---

## 8. Interview Preparation

### Key Questions to Prepare For:

1. **"Explain the difference between Ansible and Puppet"**
   - Focus on agentless vs agent-based
   - Push vs pull architecture
   - YAML vs DSL syntax

2. **"What is Infrastructure as Code?"**
   - Definition and benefits
   - Version control for infrastructure
   - Reproducible deployments

3. **"Describe a project where you used provisioning automation"**
   - Use your lab setup as an example
   - Explain the problem solved
   - Show the playbook structure

4. **"How do you handle secrets in Ansible?"**
   - Ansible Vault
   - Environment variables
   - Integration with secret management tools

5. **"What's the difference between configuration management and orchestration?"**
   - Configuration: Ansible, Puppet, Chef
   - Orchestration: Terraform, CloudFormation
   - When to use each

This comprehensive lab demonstrates practical knowledge of infrastructure automation, infrastructure as code principles, and provides concrete evidence of hands-on experience with Ansible.

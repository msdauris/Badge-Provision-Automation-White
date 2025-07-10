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

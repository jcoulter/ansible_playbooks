# ansible_playbooks
A collection of ansible playbooks 

1. Install Ansible on the machine that is provisioning your servers.

2. Make sure to configure your /etc/ansible/hosts

    ```
    [servers]                                                
    server1 ansible_host=<PROVISIONED SERVER IP> ansible_user=<DESIRED USER>  
                                                            
    [all:vars]                                               
    ansible_python_interpreter=/usr/bin/python3              
                                                            
    ```

Variables are not adequately extracted.

3. Run:
`ansible-playbook run.yml`
(may require `-kK` options for first run to prompt for passwords)
`ansible-playbook run.yml -t portainer -kK --ask-vault-password -i inventory`
hoping to someday be something like https://github.com/notthebee/infra

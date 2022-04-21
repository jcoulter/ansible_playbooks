# ansible
A collection of ansible playbooks 


initial_setup.yml
  inspiration: 
    https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-automate-initial-server-setup-on-ubuntu-20-04

  1. define server1 in /etc/ansible/hosts
  2. define created_username on line 5
  
  call: 
  ```
    ansible-playbook initial_setup.yml -l server1 -u <ssh_user> -kK
  ```

    `-k` allows ssh password
    `-K` allows sudo password

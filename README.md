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
`ansible-playbook run.yml -t frigate -kK --ask-vault-password -i inventory`
hoping to someday be something like https://github.com/notthebee/infra

brew install ansible
brew install sshpass

## Known Issues

### Tapo Camera SSL Handshake Failure (C120, C260, etc.)

Some Tapo cameras (e.g., C120) present a self-signed certificate with a weak 1024-bit RSA key (`CN=TPRI-DEVICE`)
and only support the `ECDHE-RSA-AES128-GCM-SHA256` cipher. The `kasa` library used by the TP-Link Smart Home
integration in Home Assistant doesn't offer this cipher by default, causing `SSLV3_ALERT_HANDSHAKE_FAILURE`.

**Fix:** The homeassistant ansible role patches the kasa transport layer inside the container to add the required
cipher and lower the OpenSSL security level:

```
context.set_ciphers(self.CIPHERS + ":ECDHE-RSA-AES128-GCM-SHA256:@SECLEVEL=1")
```

This patch is applied to:
- `/usr/local/lib/python3.14/site-packages/kasa/transports/sslaestransport.py`
- `/usr/local/lib/python3.14/site-packages/kasa/transports/linkietransport.py`

**Note:** The patch lives inside the container and is reapplied on every `ansible-playbook` run. If the HA base
image upgrades Python (currently 3.14), the sed path in the role will need updating.

Related issues:
- https://github.com/home-assistant/core/issues/165645
- https://github.com/home-assistant/core/issues/156926
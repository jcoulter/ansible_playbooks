# GitLab Role

This Ansible role deploys GitLab CE and GitLab Runner using Docker containers behind Traefik reverse proxy.

## Features

- GitLab CE deployment with proper Traefik integration
- GitLab Runner deployment for CI/CD
- Configurable resource limits
- Optional SMTP configuration
- Automatic network creation
- Separate data, config, and logs directories

## Requirements

- Docker
- Traefik (already configured with HTTPS)
- Sufficient system resources (GitLab requires at least 4GB RAM)

## Role Variables

See `defaults/main.yml` for all available variables and their default values.

### Important Variables

```yaml
# Domain configuration
gitlab_domain: "gitlab.example.com"
gitlab_port: "8929"  # External HTTP port
gitlab_ssh_port: "2224"  # External SSH port

# GitLab Runner
gitlab_runner_enabled: true
gitlab_runner_registration_token: "your-token-here"  # Set this after initial GitLab setup
```

## Usage

1. Include the role in your playbook:

```yaml
- hosts: servers
  roles:
    - role: services/gitlab
      tags:
        - services
        - containers
        - gitlab
```

2. Run the playbook:

```bash
ansible-playbook run.yml -t gitlab
```

3. After GitLab is deployed, retrieve the initial root password:

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

4. Log in to GitLab at https://gitlab.example.com with username `root` and the password from step 3.

5. Get the runner registration token from Admin Area -> CI/CD -> Runners.

6. Update your inventory or vars file with the token:

```yaml
gitlab_runner_registration_token: "your-token-here"
```

7. Run the playbook again to register the runner:

```bash
ansible-playbook run.yml -t gitlab
```

## Separate Runner Deployment

You can deploy or update just the GitLab Runner without touching the GitLab instance:

```bash
ansible-playbook run.yml -t gitlab-runner
```

## License

MIT

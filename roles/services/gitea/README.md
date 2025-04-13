# Gitea Role with PostgreSQL and Actions Support

This Ansible role deploys Gitea with PostgreSQL for persistent data storage and Actions (CI/CD) support for your homelab.

## Features

- Gitea server deployment using Docker
- PostgreSQL database for reliable data persistence
- Gitea Actions support for CI/CD pipelines
- Automatic runner registration
- Traefik integration for HTTPS access

## Requirements

- Docker
- Docker network named "proxy" for Traefik integration
- Traefik running as reverse proxy

## Configuration

The following variables can be configured in your inventory or vars files:

### Basic Gitea Configuration
```yaml
gitea_domain: "gitea.example.com"  # Domain for Gitea access
gitea_port: "3001"                 # Host port for Gitea HTTP
gitea_ssh_port: "23"               # Host port for Gitea SSH
gitea_data_path: "/path/to/data"   # Data storage path
gitea_config_path: "/path/to/config" # Config storage path
gitea_version: "latest"            # Gitea container version
```

### Database Configuration
```yaml
gitea_db_type: "postgres"          # Database type (postgres)
gitea_db_host: "postgres-gitea"    # PostgreSQL container name
gitea_db_port: "5432"              # PostgreSQL port
gitea_db_name: "gitea"             # Database name
gitea_db_user: "gitea"             # Database user
gitea_db_password: "gitea_password" # Database password (use ansible-vault)

postgres_data_path: "/path/to/postgres-data" # PostgreSQL data path
postgres_version: "14"             # PostgreSQL version
```

### Gitea Actions Configuration
```yaml
gitea_actions_enabled: true        # Enable Gitea Actions
gitea_actions_network_name: "gitea_network" # Docker network for Actions
gitea_runner_version: "latest"     # Runner container version
gitea_runner_name: "gitea-runner"  # Runner name
gitea_runner_data_path: "/path/to/runner-data" # Runner data path
gitea_runner_labels: "ubuntu-latest:docker://node:18-bullseye,ubuntu-22.04:docker://node:18-bullseye"
```

## Security Recommendations

1. Use Ansible Vault to encrypt sensitive information like database passwords:
   ```bash
   ansible-vault encrypt_string 'secure_password_here' --name 'gitea_db_password'
   ```

2. Add the encrypted string to your vars file or inventory.

## Runner Registration

After deploying Gitea with Actions enabled, you need to register the runner:

1. Log in to your Gitea instance as an admin
2. Go to Site Administration -> Actions -> Runners
3. Click "Create new runner" and copy the registration token
4. Add the token to your inventory or vars file:

```yaml
gitea_runner_registration_token: "your-token-here"
```

5. Re-run the playbook to register the runner

## Usage

To use Gitea Actions in your repositories:

1. Create a `.gitea/workflows/` directory in your repository
2. Add workflow files in YAML format (similar to GitHub Actions)
3. Example workflow file:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
```

## Backup Recommendations

1. Regularly backup the PostgreSQL data directory:
   ```bash
   # Example backup script
   pg_dump -U gitea -h localhost gitea > gitea_backup_$(date +%Y%m%d).sql
   ```

2. Backup the Gitea data directory which contains repositories and other files:
   ```bash
   # Example backup script
   tar -czf gitea_data_backup_$(date +%Y%m%d).tar.gz /path/to/gitea/data
   ```

## Notes

- The runner container needs access to the Docker socket to run jobs
- You can add multiple runners for better scalability
- Gitea Actions is compatible with many GitHub Actions workflows

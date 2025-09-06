# Pi-hole Ansible Role

Fully automated deployment of Pi-hole DNS server with blocklist and custom DNS configuration.

## Features

- **Complete automation** - Zero manual configuration required
- **Blocklist management** - Easily configurable blocklists with categories
- **Custom DNS entries** - Automatic local DNS resolution setup
- **Security** - Vault integration for sensitive data
- **Robust deployment** - Proper error handling and validation
- **Idempotent** - Safe to run multiple times
- **Port conflict avoidance** - Uses port 5454 to avoid systemd-resolved and mDNS conflicts

## Requirements

- Ansible 2.9+
- `community.docker` collection
- Docker and Docker Compose on target host
- Python Docker library on target host

## Installation

```bash
# Install required collection
ansible-galaxy collection install community.docker

# Add vault password (optional but recommended)
ansible-vault create vars/vault.yml
# Add: vault_pihole_admin_password: "your_secure_password"
```

## Usage

### Basic Deployment

```bash
ansible-playbook run.yml --ask-vault-password -i inventory -t pihole
```

### Configuration

**Inventory variables (host-specific):**
```yaml
[omv_server:vars]
# Base domain for your services
domain=coulter.rocks

# Specific DNS entries (for exceptions and critical services)
local_dns_entries=[{"domain": "router.coulter.rocks", "ip": "192.168.1.2"}]

# Wildcard DNS configuration (recommended for multiple services)
pihole_custom_dnsmasq_configs=[
  {
    "name": "03-wildcard-routing",
    "content": "# Router exception\naddress=/router.coulter.rocks/192.168.1.2\n\n# Wildcard for all services\naddress=/.coulter.rocks/192.168.1.36"
  }
]

# Additional blocklists (optional)
pihole_additional_blocklists=["https://custom-list.com/hosts"]
```

**Vault variables (sensitive):**
```yaml
vault_pihole_admin_password: "secure_password_here"
```

**Role variables (customize in playbook):**
```yaml
pihole_container_name: "pihole"
pihole_web_port: 8089
pihole_dns_port: 5454  # Safe port that avoids all conflicts
```

## Wildcard DNS Configuration

For multiple services on the same server, use wildcard DNS routing:

```yaml
# Route all *.coulter.rocks to your server, except router
pihole_custom_dnsmasq_configs=[
  {
    "name": "03-wildcard-routing",
    "content": "# Specific exceptions (must come first)\naddress=/router.coulter.rocks/192.168.1.2\n\n# Wildcard catch-all\naddress=/.coulter.rocks/192.168.1.36"
  }
]
```

**Benefits:**
- Automatic DNS for new services (no Pi-hole updates needed)
- Clean configuration with specific exceptions
- Works with Traefik reverse proxy setups

**Testing:**
```bash
# Test exception
dig @192.168.1.36 -p 5454 router.coulter.rocks  # → 192.168.1.2

# Test wildcard
dig @192.168.1.36 -p 5454 traefik.coulter.rocks  # → 192.168.1.36
dig @192.168.1.36 -p 5454 any-service.coulter.rocks  # → 192.168.1.36
```

## Blocklist Management

Edit `roles/pihole/vars/main.yml` to customize blocklists:

```yaml
pihole_default_blocklists:
  - "https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts"
  # Comment out to disable:
  # - "https://blocklistproject.github.io/Lists/ads.txt"
```

## File Structure

```
roles/pihole/
├── defaults/main.yml      # Default configuration
├── vars/main.yml          # Blocklist definitions
├── tasks/main.yml         # Main deployment logic
├── templates/
│   ├── docker-compose.yml.j2
│   ├── pihole.env.j2
│   └── adlists.list.j2
├── handlers/main.yml      # Restart/update handlers
└── meta/main.yml          # Role metadata
```

## Troubleshooting

**Port conflicts avoided:**
- Pi-hole runs on port 5454 instead of 53
- No conflicts with systemd-resolved (port 53) or mDNS (port 5353)
- Works alongside existing DNS and discovery services

**Collection missing:**
```bash
ansible-galaxy collection install community.docker
```

**Container not starting:**
- Check Docker service is running
- Verify port availability
- Check container logs: `docker logs pihole`

## Integration with OPNSense

After deployment, configure OPNSense:

1. **Services → Unbound DNS → General**
2. **Set Forwarding Mode: Enabled**
3. **Upstream DNS:** `192.168.1.36:5454` (your Pi-hole on port 5454)
4. **Fallback DNS:** `1.1.1.1`, `8.8.8.8`

## Ports Used

- **DNS Service:** `192.168.1.36:5454` (safe port avoiding all conflicts)
- **Web Interface:** `192.168.1.36:8086`

## Security Notes

- Admin password stored in Ansible Vault
- Environment file has restrictive permissions (0600)
- Pi-hole configuration files owned by container user
- No sensitive data in version control
- Uses non-standard DNS port for better compatibility

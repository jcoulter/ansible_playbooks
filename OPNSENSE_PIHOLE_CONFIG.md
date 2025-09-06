# OPNSense + Pi-hole High Availability Integration Guide

## Overview
Configure OPNSense with dual Pi-hole servers for high availability DNS with ad blocking and automatic failover.

## Current Architecture

### DNS Flow
```
Client → Pi-hole Primary ({{ server_ip }}:53) → OPNSense ({{ router_ip }}:53) → 1.1.1.1/8.8.8.8
        ↘ Pi-hole Backup ({{ backup_pihole_ip }}:53) ↗
```

### Components
- **Primary Pi-hole**: {{ server_ip }}:53 (OMV Server + Nebula-sync)
- **Backup Pi-hole**: {{ backup_pihole_ip }}:53 (Pi 2B - lightweight)
- **OPNSense Router**: {{ router_ip }}:53 (Upstream DNS)
- **Domain**: {{ domain }}
- **Sync**: Nebula-sync keeps both Pi-holes synchronized

## Configuration Steps

### 1. OPNSense Unbound DNS Setup

**Navigate to**: `Services → Unbound DNS → General`

#### Basic Settings
```
☑ Enable Unbound DNS Resolver
Listen Port: 53
Network Interfaces: LAN
Outgoing Network Interfaces: WAN
DNSSEC: ☑ Enable DNSSEC Support
```

#### DNS Servers Configuration
```
Server 1: 1.1.1.1@53           (Cloudflare Primary)
Server 2: 8.8.8.8@53           (Google Secondary)
Server 3: 9.9.9.9@53           (Quad9 Tertiary)
```

**Important**: Do NOT add Pi-holes here - they query OPNSense, not the reverse.

### 2. OPNSense DHCP Configuration

**Navigate to**: `Services → DHCPv4 → [LAN Interface]`

#### DNS Settings
```
DNS Servers:
  Primary: {{ server_ip }}      (Pi-hole Primary)
  Secondary: {{ backup_pihole_ip }}  (Pi-hole Backup)

Domain Name: {{ domain }}
```

### 3. Host Overrides (Local DNS)

**Navigate to**: `Services → Unbound DNS → Overrides`

#### Add Local Service Overrides
```
Host: router
Domain: {{ domain }}
Type: A
IP: {{ router_ip }}
Description: OPNSense Router

Host: pihole
Domain: {{ domain }}
Type: A
IP: {{ server_ip }}
Description: Primary Pi-hole Server

Host: pihole-backup
Domain: {{ domain }}
Type: A
IP: {{ backup_pihole_ip }}
Description: Backup Pi-hole Server
```

### 4. Advanced Configuration

**Navigate to**: `Services → Unbound DNS → Advanced`

#### Security Settings
```
☑ Hide Identity
☑ Hide Version
☑ Query Name Minimisation
☐ Strict Query Name Minimisation (can break some sites)
☑ DNS Rebind Check
```

#### Performance Settings
```
Number of Threads: 2 (or number of CPU cores)
Outgoing Range: 4096
Number of Queries per Thread: 4096
Cache Size: 100MB
Cache Max TTL: 86400
Cache Min TTL: 300
```

### 5. Access Lists (Security)

**Navigate to**: `Services → Unbound DNS → Access Lists`

#### Create LAN Access List
```
Access List Name: LAN_Networks
Action: Allow
Networks:
  - {{ server_ip | regex_replace('\.\d+$', '.0') }}/24 (your LAN subnet)
  - 127.0.0.0/8 (localhost)
Description: Allow LAN and localhost DNS queries
```

## High Availability Features

### Automatic Failover
1. **Primary Available**: Clients use {{ server_ip }} for DNS with ad blocking
2. **Primary Down**: Clients automatically use {{ backup_pihole_ip }}
3. **Both Pi-holes Down**: Clients fall back to router/ISP DNS (no blocking)

### Synchronization
- **Nebula-sync** runs on OMV server ({{ server_ip }})
- **Syncs every 5 minutes**: Blocklists, DNS entries, settings
- **Direction**: Primary → Backup (one-way sync)

### Load Distribution
- **DHCP provides both IPs**: Clients randomly choose primary/secondary
- **Natural load balancing**: Distributes queries across both Pi-holes

## Pi-hole Configuration

### Upstream DNS Settings
Both Pi-holes should be configured with:
```
Upstream DNS Servers: {{ router_ip }}#53
```

### Local DNS Entries
Both Pi-holes include:
```
router.{{ domain }} → {{ router_ip }}
*.{{ domain }} → {{ server_ip }} (wildcard for all services)
```

## Testing Configuration

### 1. Test High Availability
```bash
# Test primary Pi-hole
nslookup google.com {{ server_ip }}
nslookup doubleclick.net {{ server_ip }}  # Should be blocked (0.0.0.0)

# Test backup Pi-hole
nslookup google.com {{ backup_pihole_ip }}
nslookup doubleclick.net {{ backup_pihole_ip }}  # Should be blocked (0.0.0.0)

# Test local DNS
nslookup router.{{ domain }} {{ server_ip }}  # Should return {{ router_ip }}
nslookup test.{{ domain }} {{ server_ip }}    # Should return {{ server_ip }}
```

### 2. Test Failover
```bash
# Stop primary Pi-hole
ssh user@{{ server_ip }} "docker stop pihole"

# Test DNS still works via backup
nslookup google.com

# Restart primary Pi-hole
ssh user@{{ server_ip }} "docker start pihole"
```

### 3. Test Sync
```bash
# Add blocklist to primary Pi-hole web interface: http://{{ server_ip }}:8089/admin/
# Wait 5 minutes for nebula-sync
# Check backup Pi-hole web interface: http://{{ backup_pihole_ip }}:8089/admin/
# Blocklist should appear automatically
```

## Monitoring & Troubleshooting

### Check Pi-hole Status
```bash
# Primary Pi-hole
ssh user@{{ server_ip }} "docker ps | grep pihole"
ssh user@{{ server_ip }} "docker logs pihole --tail 10"

# Backup Pi-hole
ssh user@{{ backup_pihole_ip }} "docker ps | grep pihole"
ssh user@{{ backup_pihole_ip }} "docker logs pihole --tail 10"
```

### Check Nebula-sync Status
```bash
# Sync service (runs on primary only)
ssh user@{{ server_ip }} "docker logs nebula-sync --tail 10"
```

### OPNSense DNS Logs
**Navigate to**: `System → Log Files → Unbound DNS`

### Common Issues & Solutions

#### Issue: Backup Pi-hole Not Syncing
**Solution**:
- Check nebula-sync logs on {{ server_ip }}
- Verify both Pi-holes have same admin password
- Ensure backup Pi-hole is reachable from primary

#### Issue: DNS Queries Slow
**Solution**:
- Check Pi-hole performance on both servers
- Verify network connectivity
- Consider load balancing if one server is overloaded

#### Issue: Some Clients Not Getting Both DNS Servers
**Solution**:
- Check DHCP lease time and renewal
- Verify DHCP configuration includes both IPs
- Force DHCP renewal on affected clients

## Performance Optimization

### Recommended Settings

#### Pi-hole Configuration
```
Query Logging: Enabled (for monitoring)
Privacy Level: Show everything (for troubleshooting)
Conditional Forwarding: Enabled to {{ router_ip }}
```

#### OPNSense Cache Configuration
```
Cache Size: 100MB (for <50 devices)
Cache Size: 200MB (for 50-100 devices)
Max TTL: 86400 (24 hours)
Min TTL: 300 (5 minutes)
```

## Security Considerations

### DNS Security Features
```
☑ DNSSEC Validation (verify DNS responses)
☑ DNS Rebind Protection (prevent local network attacks)
☑ Query Name Minimisation (privacy)
```

### Access Control
- Restrict DNS queries to LAN networks only
- Monitor Pi-hole query logs for suspicious activity
- Keep Pi-hole admin interfaces on internal network only

### Backup Strategy
- **Configuration**: Nebula-sync handles Pi-hole config backup
- **OPNSense**: Regular config exports via web interface
- **Documentation**: Keep record of custom DNS overrides

## Maintenance Schedule

### Daily (Automated)
- Nebula-sync runs every 5 minutes
- Pi-hole gravity updates (if configured)

### Weekly
- Check both Pi-hole web interfaces for errors
- Verify sync is working (compare blocklist counts)
- Review DNS query logs

### Monthly
- Update Pi-hole containers if needed
- Review and optimize blocklists
- Test failover functionality

### Quarterly
- Backup OPNSense configuration
- Review DNS performance statistics
- Update documentation with any changes

## Web Interfaces

### Pi-hole Admin Interfaces
- **Primary**: http://{{ server_ip }}:8089/admin/
- **Backup**: http://{{ backup_pihole_ip }}:8089/admin/

### OPNSense Interface
- **Router**: https://{{ router_ip }}/

## Architecture Benefits

### High Availability
✅ **Redundant DNS servers** - No single point of failure  
✅ **Automatic failover** - Clients switch seamlessly  
✅ **Load distribution** - Queries spread across both servers  

### Consistency
✅ **Synchronized config** - Both Pi-holes have same blocklists  
✅ **Unified management** - Configure primary, sync to backup  
✅ **Consistent blocking** - Same ad blocking on both servers  

### Performance
✅ **Local DNS caching** - Fast response for local services  
✅ **Upstream fallback** - OPNSense handles external DNS  
✅ **Optimized routing** - Direct client → Pi-hole → router → internet

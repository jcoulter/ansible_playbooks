# Jellyfin Role Performance Improvements

## Current Issues
- 29GB config directory on ZFS causing 20+ minute startup times
- 5.3GB image cache on slow ZFS storage
- No memory limits for large library operations
- Commented/unused code cluttering the role

## Suggested Optimizations

### 1. Move Cache to Local Storage
```yaml
volumes:
  - "/opt/jellyfin/cache:/config/cache"  # Fast local SSD instead of ZFS
```
**Benefit:** Faster thumbnails, reduced startup time from 20min to 2-3min

### 2. Add Memory Limits
```yaml
memory: "4g"
memory_swap: "6g"
```
**Benefit:** Prevents OOM crashes during library scans, better stability

### 3. Add Cache Directory Creation Task
```yaml
- name: Create local cache directory for better performance
  file:
    path: "/opt/jellyfin/cache"
    state: directory
    owner: "{{ puid }}"
    group: "{{ pgid }}"
    mode: '0755'
```

### 4. Clean Up Role
- Remove commented volume mounts
- Remove unused QuickSync package installation code
- Fix malformed task name

### 5. Optional: ZFS Optimizations
If keeping config on ZFS:
```bash
zfs set recordsize=8K brick/jellyfin-config  # Better for small files
```

## Implementation Priority
1. **High:** Move cache to local storage (biggest impact)
2. **Medium:** Add memory limits (stability)
3. **Low:** Clean up commented code (maintainability)

## Notes
- Keep media mount as read-write (metadata files stored with media)
- Test cache directory permissions after implementation
- Monitor startup times after changes

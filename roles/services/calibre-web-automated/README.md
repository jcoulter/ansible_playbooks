# Calibre-Web-Automated Role

Deploys two instances of [Calibre-Web-Automated](https://github.com/crocodilestick/Calibre-Web-Automated):

- **Primary** (`ebooks.coulter.rocks`) — main ebook library
- **Alt** (`ebooks-alt.coulter.rocks`) — secondary library

## Volumes

| Container Path | Host Path | Purpose |
|---------------|-----------|---------|
| `/config` | `{{ config_root }}/calibre_web_automated_config` | App database, settings |
| `/cwa-book-ingest` | `{{ data_root }}/media/cwa_books/ingest` | Drop books here for auto-import |
| `/calibre-library` | `{{ data_root }}/media/cwa_books/library` | Calibre library (managed by CWA) |

## Usage

Drop `.epub`, `.pdf`, `.mobi`, or other ebook files into the ingest folder. CWA will automatically import them into the Calibre library with metadata and covers.

## Known Issue: File Permissions

**Problem:** CWA's internal Calibre subprocess runs as root with umask `022`, ignoring the `UMASK` and `PUID`/`PGID` environment variables. Newly ingested files get `644` permissions, which prevents editing metadata through the web UI.

**Upstream issue:** https://github.com/crocodilestick/Calibre-Web-Automated/issues/30

**Fix:** This role deploys a systemd path watcher (`cwa-fix-perms.path`) on the host that:

1. Watches the library directory for modifications (file creation/changes)
2. Triggers `cwa-fix-perms.service` which waits 5 seconds for CWA to finish writing
3. Sets group-write permissions and setgid bits on all library files/directories

This is event-driven (no polling), runs on the host (survives container redeploys), and is fully managed by this ansible role.

### Manual permission fix

If you need to fix permissions immediately without waiting for the watcher:

```bash
sudo chmod -R g+w /mnt/data/media/cwa_books/library/
sudo find /mnt/data/media/cwa_books/library -type d -exec chmod g+s {} \;
```

## Tags

```bash
ansible-playbook run.yml --ask-vault-password -i inventory -t calibre-web-automated
```

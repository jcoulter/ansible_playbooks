
There is a bug preventing nextcloud from working over nfs share because of locking. Not sure how I am going to store this on the nas, would samba share work?

Both docker-compose.yml and the nextcloud role are using local volumes.

- ipv6
- get nextcloud saving on nas
- get swag reverse proxy working
- I'd like to get jellyfin running. The current config requires swag

- get swag reverse proxy working

Add docs app


Add notes app
https://trilium.cc/
https://community.hetzner.com/tutorials/install-trilium-notes-using-docker

https://www.linuxjournal.com/content/my-love-affair-synology


syncthing on nas can share roms and save files accross multiple retro pies


duplicati to make encrypted remote backups?

# intel graphics drivers

https://www.reddit.com/r/jellyfin/comments/wodeaj/celeron_n5105_hardware_transcoding/
https://gist.github.com/Brainiarc7/aa43570f512906e882ad6cdd835efe57
https://blogs.igalia.com/vjaquez/2017/12/07/enabling-huc-for-sklkbl-in-debiantesting/

sudo apt-get install xserver-xorg-video-intel


Make docker tasks add the ExecStartPre line to 
 /lib/systemd/system/docker.service
 
```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket containerd.service

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStartPre=/bin/sleep 30
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target

```


Xinstall tdarr
Xcalibre?

update tdarr node
update tvshows dir
frigate wyze
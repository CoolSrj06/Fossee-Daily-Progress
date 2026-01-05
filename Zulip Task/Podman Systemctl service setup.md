## Initial status

![alt text](<./images/Pasted image 20260102162509.png>)

Containers are Up and Running. A docker compose file that does this building of the containers

![[Pasted image 20260102162600.png]]

## Creating a Systemd service

+ ### Step 1
Stop all the running containers.
```bash
podman compose down -v
```

+ ### Step 2
```bash
# Run the below command in docker-zulip directory
 podman compose systemd -a create-unit
```

+ ### Step 3
Create a file at  `vi ~/.config/systemd/user/podman-compose@docker-zulip.service`. 
```bash
# Save this in that file
[Unit]
Description=%i rootless pod (podman-compose)

[Service]
Type=simple
EnvironmentFile=%h/.config/containers/compose/projects/%i.env
ExecStartPre=-/home/zmaurya/.local/bin/podman-compose up --no-start
ExecStartPre=/usr/bin/podman pod start pod_%i
ExecStart=/home/zmaurya/.local/bin/podman-compose wait
ExecStop=/usr/bin/podman pod stop pod_%i

[Install]
WantedBy=default.target
```

+ ## Step 4
```bash
# Relaod and enable service file 
systemctl --user daemon-reload
systemctl --user enable podman-compose\@docker-zulip.service
systemctl --user start podman-compose\@docker-zulip.service
# The service must start successfully
systemctl --user status podman-compose\@docker-zulip.service
```

## Conclusion

The containers will start to run and thus now you service functionable.

# ssh-agent-systemd

## systemd

Create a systemd service by creating `~/.config/systemd/user/ssh-agent.service` with following contents:
```
[Unit]
Description=SSH key agent

[Service]
Type=simple
NoNewPrivileges=yes
PrivateTmp=yes
DevicePolicy=closed
ProtectSystem=strict
ProtectHome=read-only
RestrictNamespaces=yes
RestrictRealtime=yes
MemoryDenyWriteExecute=yes
LockPersonality=yes
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -t 14400 -a ${SSH_AUTH_SOCK}

[Install]
WantedBy=default.target
```

Tell systemd about the new unit:
```
systemctl --user daemon-reload
```

Enable and start the service:
```
systemctl --user enable ssh-agent
systemctl --user start ssh-agent
```

Add to bashrc:
```
echo "export SSH_AUTH_SOCK=${XDG_RUNTIME_DIR}/ssh-agent.socket" >> ~/.bashrc
```

Consider further restricting the service for security. You can make use of systemd's analyze functionality:
```
systemd-analyze security --user ssh-agent
```

View service logs/journal output:
```journalctl --user -u ssh-agent```
or
```journalctl --user-unit ssh-agent```


## ssh-agent

To list keys currently stored in the agent:
```
ssh-add -l
```

or, for full public keys:
```
ssh-add -L
```

Add key manually to agent:
```
ssh-add /path/to/key
```


## ssh config

To automatically have keys added to your agent use the following directive (OpenSSH 7.2+) in your ssh config:
```
echo 'AddKeysToAgent yes' >> ~/.ssh/config
```

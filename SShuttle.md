# SShuttle

#tech-tip

https://github.com/sshuttle/sshuttle

Useful for airgapped deployments. Use SSHuttle to route all traffic through an SSH tunnel.

For K3S:
```bash
sshuttle -r ubuntu@3.73.45.151 0/0 -x 3.73.45.151 -x 10.43.0.0/16 -x 10.42.0.0/24
```
Where:
- 3.73.45.151 - ssh proxy IP
- `-x 3.73.45.151` - exclude connections to the proxy IP from being routed through the proxy
- `-x 10.43.0.0/16` - exclude **POD CIDR** from being routed through the proxy
- `-x 10.42.0.0/24`- exclude **SERVICE CIDR**.

## Auto Startup

- Create service fiule:
```bash
sudo vim /etc/systemd/system/sshuttle.service
```
- Paste:
```
[Unit]
Description=SSHuttle VPN
After=network.target

[Service]
User=root
ExecStart=/usr/bin/sshuttle -r ubuntu@3.73.45.151 0/0 -x 3.73.45.151 -x 10.43.0.0/16 -x 10.42.0.0/24
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

- Reload:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sshuttle.service
sudo service sshuttle restart
```
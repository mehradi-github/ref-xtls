# Installign XTLS on Linux
XTLS protocol, providing a set of network tools such as Xray-core and REALITY.

- [Installign XTLS on Linux](#installign-xtls-on-linux)
  - [Server](#server)
    - [Protect your server with iptables](#protect-your-server-with-iptables)
    - [Setting kernel for performance and raise ulimits](#setting-kernel-for-performance-and-raise-ulimits)
    - [Install Xray](#install-xray)
    - [Determining camouflage website](#determining-camouflage-website)
    - [Adding Xray server's config](#adding-xray-servers-config)
    - [Enabling and starting the Xray service](#enabling-and-starting-the-xray-service)
  - [client](#client)
    - [Adding Xray client's config](#adding-xray-clients-config)
## Server
### Protect your server with iptables
```sh
# replacing <HOME-IP-ADDRESS>
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -s <HOME-IP-ADDRESS> -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -P INPUT DROP

ip6tables -A INPUT -i lo -j ACCEPT
ip6tables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
ip6tables -A INPUT -p ipv6-icmp -j ACCEPT
ip6tables -P INPUT DROP

# Make the iptables rules permanent
sudo apt install iptables-persistent
```
### Setting kernel for performance and raise [ulimits](https://phoenixnap.com/kb/ulimit-linux-command)
```sh
# performance
sudo cat <<EOF > /etc/sysctl.d/xray-sysctl.conf
net.ipv4.tcp_keepalive_time = 90
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_fastopen = 3
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
fs.file-max = 65535000
EOF
# ulimits
sudo cat <<EOF > /etc/security/limits.d/xray-limit.conf
* soft     nproc          655350
* hard     nproc          655350
* soft     nofile         655350
* hard     nofile         655350
root soft     nproc          655350
root hard     nproc          655350
root soft     nofile         655350
root hard     nofile         655350
EOF

sudo sysctl --system
```

### Install Xray

```sh
sudo apt update && Sudo apt upgrade

# Install Xray version 1.8.1 to run as root
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root --version 1.8.1

# Generate parameters
# UUID
xray uuid -i Secret
# Private and Public keys
xray x25519
# short ID
openssl rand -hex 8


```
### Determining camouflage website
- Be a foreign website
- Support TLSv1.3 and H2
- Have a URL that is not redirected elsewhere (though the apex domain name may be redirected to www)
- Bonus points if it has a similar IP to your server
  
### Adding Xray server's config
You can see some [Xray-examples](https://github.com/XTLS/Xray-examples) of server config.json for Xray-core.

Download [config.json](https://github.com/mehradi-github/ref-xtls/blob/main/configs/config_server.json) and edit params like as below:

```json
{
   //...
    "inbounds": [
        {
            "listen": "0.0.0.0",  // "0.0.0.0" Indicates listening to both IPv4 and IPv6
            "port": 443, // The port on which the server listens
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "EDIT-UUID", // Your generated UUID here.
                        "flow": "xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false,
                    "dest": "EDIT-DEST", // ex: www.microsoft.com:443
                    "xver": 0,
                    "serverNames": [
                        "EDIT-SERVERNAME" //ex: www.microsoft.com
                    ],
                    "privateKey": "EDIT-PRIVATEKEY", // Private key you generated earlier.
                    "minClientVer": "",
                    "maxClientVer": "",
                    "maxTimeDiff": 0,
                    "shortIds": [
                        "EDIT-SHORTID" // Short ID
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            }
        }
    ],
        
  //...
}        
```
after that save in **/usr/local/etc/xray/config.json**

### Enabling and starting the Xray service
```sh
# sudo cat <<EOF > /etc/systemd/system/xray.service
# [Unit]
# Description=XTLS Xray-Core a VMESS/VLESS Server
# After=network.target nss-lookup.target
# [Service]
# User=USERNAME
# Group=USERNAME
# CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
# AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
# NoNewPrivileges=true
# ExecStart=/home/USERNAME/xray/xray run -config /home/USERNAME/xray/config.json
# Restart=on-failure
# RestartPreventExitStatus=23
# StandardOutput=journal
# LimitNPROC=100000
# LimitNOFILE=1000000
# [Install]
# WantedBy=multi-user.target
# EOF

sudo systemctl daemon-reload && sudo systemctl enable xray
sudo systemctl start xray && sudo systemctl status xray
# show logs
journalctl -u xray | less


```
## client
 ### Adding Xray client's config

You can see some [Xray-examples](https://github.com/XTLS/Xray-examples) of client config.json for Xray-core.

Download [config.json](https://github.com/mehradi-github/ref-xtls/blob/main/configs/client-config.json) and edit params like as below:

```json
{
//...
"outbounds": [
        {
            "protocol": "vless",
            "settings": {
                "vnext": [
                    {
                        "address": "EDIT-ADDRESS", // IP server or DNS
                        "port": 443,
                        "users": [
                            {
                                "id": "EDIT-UUID", // Your generated UUID here.
                                "flow": "xtls-rprx-vision",
                                "encryption": "none"
                            }
                        ]
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    "show": false,
                    "fingerprint": "chrome",
                    "serverName": "EDIT-SERVERNAME", //ex: www.microsoft.com
                    "publicKey": "EDIT-PUPLICKEY",  // Public key you generated earlier.
                    "shortId": "EDIT-SHORTID", // Short ID
                    "spiderX": "/"
                }
            },
            "tag": "proxy"
        },
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ]

//...
}
```

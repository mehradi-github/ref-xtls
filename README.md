# Installign XTLS on Linux
XTLS protocol, providing a set of network tools such as Xray-core and REALITY.

- [Installign XTLS on Linux](#installign-xtls-on-linux)
  - [Protect your server with iptables](#protect-your-server-with-iptables)
  - [Setting kernel for performance and raise ulimits](#setting-kernel-for-performance-and-raise-ulimits)
  - [Install Xray](#install-xray)
  - [Adding xray's config](#adding-xrays-config)
  - [Enabling Xray.service](#enabling-xrayservice)

## Protect your server with iptables
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
## Setting kernel for performance and raise [ulimits](https://phoenixnap.com/kb/ulimit-linux-command)
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

## Install Xray

```sh
sudo apt update && Sudo apt upgrade

# Install Xray version 1.8.1 to run as root
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root --version 1.8.1


# sudo apt install vim curl unzip
# curl -fsSLO https://github.com/XTLS/Xray-core/releases/download/v1.8.1/Xray-linux-64.zip
# unzip Xray-linux-64.zip -d ~/xray && cd ~/xray

# Generate parameters
# UUID
./xray uuid -i Secret
# Private and Public keys
./xray x25519
# short ID
openssl rand -hex 8


```

## Adding xray's config
```sh
cd ~/xray
curl -fsSLO https://github.com/mehradi-github/ref-xtls/blob/main/config.json
```

```json
{
         "listen":"0.0.0.0",
         "port":443,
         "protocol":"vless",
         "settings":{
            "clients":[
               {
                  "id":"EDIT-UUID", // Your generated UUID here.
                  "flow":"xtls-rprx-vision"
               }
            ],
            "decryption":"none"
         },
         "streamSettings":{
            "network":"tcp",
            "security":"reality",
            "realitySettings":{
               "show":false,
               "dest":"EDIT-DEST", // ex: www.google-analytics.com:443 Edit to a website/server that works without VPN outside of Iran
               "xver":0,
               "serverNames":[
                  "EDIT-SERVERNAME" // ex: www.google-analytics.com (SNI) Same as "dest" but without portnumber. 
               ],
               "privateKey":"EDIT-PRIVATEKEY", // Private key you generated earlier.
               "minClientVer":"1.8.0",
               "maxClientVer":"",
               "maxTimeDiff":0,
               "shortIds":["EDIT-SHORTID" // Short ID
               ]
            }
         },
  //...
}        
```
## Enabling Xray.service
```sh
# Changing USERNAME to your username 
sudo cat <<EOF > /etc/systemd/system/xray.service
[Unit]
Description=XTLS Xray-Core a VMESS/VLESS Server
After=network.target nss-lookup.target
[Service]
User=USERNAME
Group=USERNAME
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/home/USERNAME/xray/xray run -config /home/USERNAME/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
StandardOutput=journal
LimitNPROC=100000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload && sudo systemctl enable xray

sudo systemctl start xray && sudo systemctl status xray

journalctl -u xray | less


```


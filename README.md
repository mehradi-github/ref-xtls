# Installign XTLS on Linux
XTLS protocol, providing a set of network tools such as Xray-core and REALITY.

- [Installign XTLS on Linux](#installign-xtls-on-linux)
  - [Setting kernel for performance and raise ulimits](#setting-kernel-for-performance-and-raise-ulimits)
  - [Install Xray (XTLS)](#install-xray-xtls)


## Setting kernel for performance and raise [ulimits](https://phoenixnap.com/kb/ulimit-linux-command)
```sh
# performance
sudo cat <<EOF > /etc/sysctl.d/xtls.conf
net.ipv4.tcp_keepalive_time = 90
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_fastopen = 3
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
fs.file-max = 65535000
EOF
# ulimits
sudo cat <<EOF > /etc/security/limits.d/xtls.conf
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

## Install Xray (XTLS)

```sh
sudo apt update && Sudo apt upgrade
sudo apt install curl unzip

curl -fsSLO https://github.com/XTLS/Xray-core/releases/download/v1.8.1/Xray-linux-64.zip
unzip Xray-linux-64.zip -d ~/xray && cd ~/xray

# Generateing UUID, (Private and Public) keys, short IDs
./xray uuid -i Secret
./xray x25519
openssl rand -hex 8







```


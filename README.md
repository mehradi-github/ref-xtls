# XTLS
XTLS protocol, providing a set of network tools such as Xray-core and REALITY.

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

sudo sysctl -p
```


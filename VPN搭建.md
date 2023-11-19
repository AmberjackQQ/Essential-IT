# VPN简述

VPN是虚拟私有网络Virtual Private Network的简称，它能在不可信的网络上建立私有的网络。它能让你自由、安全的连接从你的各种终端，例如手机、平板电脑等等。OpenVPN 是一套实现VPN的综合解决方案，它功能齐全并且安全高效，代码开源。

## OpenVPN server 安装步骤, 以在CentOS 7上为例
1. 安装epel源

    `yum install epel-release -y`

2. 安装openvpn

    `yum install -y openvpn`

3. 安装Easy RSA 
```bash
    `yum install -y openvpn wget`
    `wget -O /tmp/easyrsa https://github.com/OpenVPN/easy-rsa-old/archive/2.3.3.tar.gz`
    `tar xfz /tmp/easyrsa`
    `mkdir /etc/openvpn/easy-rsa`
    `cp -rf easy-rsa-old-2.3.3/easy-rsa/2.0/* /etc/openvpn/easy-rsa`
```
4. 配置OpenVPN

        指定VPN使用端口，IP地址范围，route的范围以及DHCP, 详情参见[server.conf](./openvpnconf/server.conf)

5. 建立密钥和证书

    `mkdir /etc/openvpn/easy-rsa/keys`
    
    编辑 [/etc/openvpn/easy-rsa/vars](./openvpnconf/vars)
```
    cd /etc/openvpn/easy-rsa
    source ./vars
    ./clean-all
    cp openssl-1.0.0.cnf openssl.cnf
    ./build-ca
    ./build-key-server server
    ./build-dh
    cd /etc/openvpn/easy-rsa/keys
    cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
    cd /etc/openvpn/easy-rsa
    ./build-key client
    openvpn --genkey --secret myvpn.tlsauth
```

5. 配置防火墙

```
        firewall-cmd --get-active-zones， 确认tun0在信任区
        firewall-cmd --zone=trusted --add-service openvpn
        firewall-cmd --zone=trusted --add-service openvpn --permanent
        firewall-cmd --list-services --zone=trusted
        firewall-cmd --add-masquerade
        firewall-cmd --permanent --add-masquerade
        firewall-cmd --query-masquerade
        SHARK=$(ip route get 8.8.8.8 | awk 'NR==1 {print $(NF-2)}')
        firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o $SHARK -j MASQUERADE
        增加 net.ipv4.ip_forward = 1 到 /etc/sysctl.conf
        sysctl --system
        systemctl restart network.service
```
6. 配置daemon服务

```
        systemctl -f enable openvpn@server.service
        systemctl start openvpn@server.service
        systemctl status openvpn@server.service 
```

## OpenVPN client 安装步骤， 
1. 从openvpn 服务器获取如下文件，放在/etc/openvpn/client/目录下
    - /etc/openvpn/easy-rsa/keys/ca.crt
    - /etc/openvpn/easy-rsa/keys/client.crt
    - /etc/openvpn/easy-rsa/keys/client.key
    - /etc/openvpn/myvpn.tlsauth

    编辑[1.conf](./openvpnconf/1.conf)，看起来像如下 
```    
    client
    tls-client
    ca /etc/openvpn/client/ca.crt
    cert /etc/openvpn/client/client.crt
    key /etc/openvpn/client/client.key
    tls-crypt /etc/openvpn/client/myvpn.tlsauth
    ;remote-cert-eku "TLS Web Client Authentication"
    proto udp
    remote <openvpn服务器地址> 1194 udp
    dev tun
    topology subnet
    pull
    verb 3
```    
2.  设置openvpn-client服务
    `cp /usr/lib/systemd/system/openvpn-client\@.service /usr/lib/systemd/system/openvpn-client\@1.service `
    `systemctl enable openvpn-client\@1`
3.  检查服务是否成功
    `systemctl status openvpn-client\@1`
4.  检查路由器
    `route -n`

# 迁移openvpn
1. 搭建一台新的CentOS7服务器,硬件如下,安装centos7.9 server with GUI 和openvepn
   80processor Intel(R) Xeon(R) Gold 6138 CPU @ 2.00GHz
   dmidecode -t memory | grep  Size: | grep -v "No Module Installed" | awk '{sum+=$2}END{print sum}'
   768GB
2. 用nc 工具选择合适的协议和规则，tcp 443
    nc -u -l 1194
    nc <openvpn server IP> 1194
3. 配置网卡到trusted zone 并且设置 NAT, ip_forward=1
    firewall-cmd --permanent --zone=trust --add-interface=eno1    firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.1.0/24 -o eno1 -j MASQUERADE
    firewall-cmd --reload
    systemctl restart firewalld
4. 复制/etc/openvpn
5. 启动服务openvpn@server

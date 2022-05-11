# Essential-IT
# 必备技能一，搭建VPN
VPN是虚拟私有网络Virtual Private Network的简称，它能在不可信的网络上建立私有的网络。它能让你自由、安全的连接从你的各种终端，例如手机、平板电脑等等。
A Virtual Private Network (VPN) allows you to traverse untrusted networks as if you were on a private network. It gives you the freedom to access the internet safely and securely from your smartphone or laptop when connected to an untrusted network, like the WiFi at a hotel or coffee shop.

When combined with HTTPS connections, this setup allows you to secure your wireless logins and transactions. You can circumvent geographical restrictions and censorship, and shield your location and any unencrypted HTTP traffic from the untrusted network.

OpenVPN is a full featured, open-source Secure Socket Layer (SSL) VPN solution that accommodates a wide range of configurations. In this tutorial, you will set up OpenVPN on a CentOS 7 server, and then configure it to be accessible from a client machine.

Note: If you plan to set up an OpenVPN server on a DigitalOcean Droplet, be aware that we, like many hosting providers, charge for bandwidth overages. For this reason, please be mindful of how much traffic your server is handling.

See this page for more info.

1.安装epel源
yum install epel-release -y
2.安装openvpn
yum install -y openvpn
3.从openvpn 服务器获取 
    client.crt client.key 1.conf  myvpn.tlsauth ca.crt
    放在/etc/openvpn/client/目录下
    编辑1.conf，看起来像如下
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
4.设置openvpn-client服务
mv /usr/lib/systemd/system/openvpn-client\@.service /usr/lib/systemd/system/openvpn-client\@1.service 
systemctl enable openvpn-client\@1
5.检查服务是否成功
systemctl status openvpn-client\@1 
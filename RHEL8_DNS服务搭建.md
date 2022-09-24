# 安装步骤
1. 同步系统时间
2. 进入yum.repos.d, 注销mirrorlist 添加
    baseurl=http://vault.centos.org/$contentdir/$releasever/BaseOS/$basearch/os/ 在BaseOS
    baseurl=http://vault.centos.org/$contentdir/$releasever/AppStream/$basearch/os/ 在Appstream
3. yum install bind-chroot -y
4. 编辑/etc/named.conf
    listen-on port 53 { any; };
    allow-query     { any; };
5. 编辑/etc/named.rfc1912.zones
 zone "skp.veritas.com" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};
6. cp /var/named/named.localhost /var/named//var/named/named.localhost.bk
7. 编辑/var/named/named.localhost
$TTL 1D
@       IN SOA  skp.veritas.com. root.skp.veritas.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
www     IN      A       10.200.39.73
mail    IN      A       10.200.39.3
8. 编辑named.loopback
@       IN SOA  ns.skp.veritas.com. root.skp.veritas.com (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
73      IN      PTR     www.skp.veritas.com
3       IN      PTR     mail.skp.veritas.com
9. 检验两个配置文件named-checkzone skp.veritas.com /var/named/named.loopback named.localhost
10. 配置SElinux 和 firewalld
[root@localhost yum.repos.d]# getsebool -a | grep named
named_tcp_bind_http_port --> off
named_write_master_zones --> on
[root@localhost yum.repos.d]# setsebool -P named_tcp_bind_http_port
[root@localhost yum.repos.d]# setsebool -P named_tcp_bind_http_port on
[root@localhost yum.repos.d]# getsebool -a | grep named
named_tcp_bind_http_port --> on
named_write_master_zones --> on
firewall-cmd --add-port=53/tcp --permanent
firewall-cmd --add-service=dns --permanent
systemctl restart named.service
- 安装tigervnc-server
    yum install -y tigervnc-server
- 配置 user.map :1=root
- 启动和开启服务 systemctl enable vncserver@:1
- 客户端测试 ssh -C -L localhost:5901:10.200.xx.xx:5901 root@10.200.xx.xx
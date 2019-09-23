## CentOS7搭建OpenVPN记录
### 1. 关闭SELinux  
```bash
vi /etc/selinux/config
```
### 2. 关闭firewalld  
```bash
systemctl disable firewalld
systemctl stop firewalld
```

### 3. 安装openVPN  
```bash
yum install epel-release
yum update
yum -y install openvpn easy-rsa
```
证书创建参考easy-rsa中的README  

allpass模式下可能需要修改DNS服务器（否则类似192.168.1.1这种DNS访问不到），server.conf中去掉下面代码的注释  
```conf
push "dhcp-option DNS 203.112.2.4"
push "dhcp-option DNS 203.112.2.5"
```
如要开启ipv6监听则加入代码
```conf
proto udp6
```
将server.conf和证书等文件放到/etc/openvpn/中(样本配置在/usr/share/doc/openvpn-2.4.6/sample/sample-config-files/server.conf)  
```bash
systemctl start openvpn@server
systemctl enable openvpn@server
```
### 4. 开启IP转发  
```bash
cat /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
```
### 5. 配置NAT  
```bash
iptables --table nat --append POSTROUTING --out-interface ens192 --jump MASQUERADE
```
### 6. 客户端设置
windows虚拟机allpass模式连不上Internet解决方式：1. 连接VPN； 2. 打开网络共享； 3. 断开VPN； 4. 自动诊断修复以太网卡（诊断的原因是此网卡未启用DHCP）； 5. 连接VPN； 6. 关闭网络共享。  
笔记本上的windows经测试allpass模式没有上面的问题  
windows虚拟机连接上allpass模式后无法公网ip访问解决方式：1. 连接VPN； 2. 打开网络共享； 3. 关闭网络共享。  

如果客户端是linux deb系需要利用/etc/openvpn/update-resolv-conf脚本修改resolv.conf需安装
```bash
apt-get install resolvconf
```
之后在client.conf最后加三行
```conf
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf  
```

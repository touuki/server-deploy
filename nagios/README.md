# Nagios
监控
## Nagios Core
安装`yum -y install nagios`，配置文件`/etc/nagios/nagios.cfg`中加入相应配置文件
```cfg
cfg_file=/etc/nagios/objects/commands.cfg
cfg_file=/etc/nagios/objects/contacts.cfg
cfg_file=/etc/nagios/objects/timeperiods.cfg
cfg_file=/etc/nagios/objects/templates.cfg
cfg_file=/etc/nagios/objects/hostgroups.cfg
cfg_file=/etc/nagios/objects/samplehost.cfg
```

启动服务，设置开机服务
```bash
systemctl start nagios
systemctl enable nagios
```
## Nagios Plugins
安装`yum -y install nagios-plugins-all`，将下列自定义脚本放至`/usr/lib64/nagios/plugins/`，注意给予执行权限`chmod +x`
* [check_mem](https://exchange.nagios.org/directory/Plugins/Operating-Systems/Linux/check_mem/details)
* [check_iostat](https://exchange.nagios.org/directory/Plugins/Operating-Systems/Linux/check_iostat--2D-I-2FO-statistics--2D-updated-2016/details)
* [check_traffic](https://exchange.nagios.org/directory/Plugins/Network-Connections%2C-Stats-and-Bandwidth/check_traffic-2Esh/details)/[github](https://github.com/cloved/check_traffic)
* [html_email](https://exchange.nagios.org/directory/Plugins/Notifications/Responsive-HTML-Email-Notifications-Templates-for-Nagios/details)/[github](https://github.com/heiniha/Nagios-Responsive-HTML-Email-Notifications)

## NRPE
### On Nagios Host
安装check_nrpe，`yum install nagios-plugins-nrpe`

### On Monitored Host
安装nrpe daemon，`yum install nrpe`，修改`/etc/nagios/nrpe.cfg`
```cfg
allowed_hosts=127.0.0.1,::1,nagios.belew.tech

command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib64/nagios/plugins/check_load -r -w 1.0,0.9,0.8 -c 1.3,1.15,1.0
command[check_disk]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/vda1 # -C -w 20 -c 10 -p /dev/vdb1 -u GiB
command[check_zombie_procs]=/usr/lib64/nagios/plugins/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 300 -c 500
command[check_iostat]=/usr/lib64/nagios/plugins/check_iostat -d vda1 -w 1200,84000,84000,50 -c 2000,96000,96000,100
command[check_mem]=/usr/lib64/nagios/plugins/check_mem -w 80% -c 90%
command[check_traffic]=/usr/lib64/nagios/plugins/check_traffic -V 2c -H localhost -C local -N eth0 -w 80000,16000 -c 90000,18000 
command[check_mysql]=/usr/lib64/nagios/plugins/check_mysql -H localhost -u username -p password

command[check_iostat_ssd]=/usr/lib64/nagios/plugins/check_iostat -d vdb1 -w 16000,240000,240000,50 -c 18000,270000,270000,100
```

启动服务，设置开机服务
```bash
systemctl start nrpe
systemctl enable nrpe
```
注意防火墙对Nagios Host开放5666端口

## SNMP
安装[Net-SNMP](http://www.net-snmp.org/)，`yum -y install net-snmp`，修改`/etc/snmp/snmpd.conf`
```cfg
# Here is a commented out example configuration that allows less
# restrictive access.

# YOU SHOULD CHANGE THE "COMMUNITY" TOKEN BELOW TO A NEW KEYWORD ONLY
# KNOWN AT YOUR SITE.  YOU *MUST* CHANGE THE NETWORK TOKEN BELOW TO
# SOMETHING REFLECTING YOUR LOCAL NETWORK ADDRESS SPACE.

##       sec.name  source          community
com2sec local     localhost       local
#com2sec mynetwork NETWORK/24      COMMUNITY

##     group.name sec.model  sec.name
group MyRWGroup  any        local
#group MyROGroup  any        mynetwork
#
#group MyRWGroup  any        otherv3user
#...

##           incl/excl subtree                          mask
view all    included  .1                               80

## -or just the mib2 tree-

#view mib2   included  .iso.org.dod.internet.mgmt.mib-2 fc

##                context sec.model sec.level prefix read   write  notif
#access MyROGroup ""      any       noauth    0      all    none   none
access MyRWGroup ""      any       noauth    0      all    all    all
```
启动服务，设置开机启动
```bash
systemctl start snmpd
systemctl enable snmpd
```

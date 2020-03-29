## Nagios告警通知设置

### 1、安装mailx并配置nagios的邮箱配置文件：
```ruby
# yum install mailx*
# vim /etc/mail.rc
set from=13662097373@163.com
set smtp=smtp.163.com
set smtp-auth-user=13662097373@163.com
set smtp-auth-password=29755396Wzy             #授权码
set smtp-auth=login
```
```ruby
# vim /usr/local/nagios/etc/objects/templates.cfg
define contact{
        name                            generic-contact        
        service_notification_period     24x7                   
        host_notification_period        24x7                  
        service_notification_options    w,u,c,r,f,s           
        host_notification_options       d,u,r,f,s              
        service_notification_commands   notify-service-by-email
        host_notification_commands      notify-host-by-email   
        register                        0                      
        }


define host{
        name                            generic-host    
        notifications_enabled           1              
        event_handler_enabled           1              
        flap_detection_enabled          1              
        failure_prediction_enabled      1             
        process_perf_data               1             
        retain_status_information       1              
        retain_nonstatus_information    1             
        notification_period             24x7         
        register                        0              
        }

define host{
        name                            linux-server   
        use                             generic-host   
        check_period                    24x7           
        check_interval                  5              
        retry_interval                  1              
        max_check_attempts              10          
        check_command                   check-host-alive
        notification_period             workhours      
                                                    
        notification_interval           120            
        notification_options            d,u,r  
        contact_groups                  admins        
        register                        0             
        }
//max_check_attempts定义为1，检测到问题后立即报警，不重试。
// notification_interval：重复发送提醒信息的最短间隔时间。默认间隔时间是60分钟。如果这个值设置为0，将不会发送重复提醒。
```
```ruby
# vim /usr/local/nagios/etc/objects/commands.cfg
# 'notify-host-by-email' command definition
define command{
   command_name   notify-host-by-email
   command_line   /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRSS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mailx -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
   }

# 'notify-service-by-email' command definition
 define command{
   command_name       notify-service-by-email
   command_line      /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mailx -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
   }
```
```ruby
# vim /usr/local/nagios/etc/objects/contacts.cfg
define contact{
        contact_name                    nagiosadmin           
        use                             generic-contact        
        alias                           Nagios Admin        
        host_notifications_enabled      1
        service_notifications_enabled   1
        email                           479414941@qq.com      
        }


define contactgroup{
        contactgroup_name       admins
        alias                   Nagios Administrators
        members                 nagiosadmin
        }
```
```ruby
# vim /usr/local/nagios/etc/objects/linux.cfg
define host{
        use             linux-server,host-pnp
        host_name       mysql1
        alias           My Linux Server
        address         192.168.1.112
        }

```
### 2、测试
```ruby
# echo "hello word" | mailx -s "mail title" 479414941@qq.com
设置延时：
# vim /usr/local/nagios/etc/nagios.cfg
notification_timeout=300
参数：
# vim /usr/local/nagios/etc/objects/templates.cfg
host_notification_options：
d = notify on DOWN host states,
u = notify on UNREACHABLE host states
r = notify on host recoveries (UP states)
f = notify when the host starts and stops flapping
s = send notifications when host or service scheduled downtime starts and ends
n (none) as an option, the contact will not receive any type of host notifications.
service_notification_options:
w = notify on WARNING service states
u = notify on UNKNOWN service states
c = notify on CRITICAL service states
r = notify on service recoveries (OK states)
f = notify when the service starts and stops flapping
n (none) as an option, the contact will not receive any type of service notifications.

常用的设置
host_notification_options：d,u,r
service_notification_options:w,u,c,r
```
### 3、设置告警次数:
```ruby
vi /usr/local/nagios/etc/objects/escalations.cfg
define serviceescalation{
host_name                    192.168.1.1      ;被监控主机名称，多个用逗号隔开与Hosts.cfg中一致
service_description        SSH                ;被监控服务名称，多个用逗号隔开 与services.cfg中一致
first_notification           4                         ; 第4条信息起，改变频率间隔
last_notification            0                        ; 第n条信息起，恢复频率间隔
notification_interval        30                    ; 通知间隔(单位：分)
contact_groups          admins
}

define serviceescalation{
host_name                  192.168.1.1
service_description        SSH
first_notification           10
last_notification            0
notification_interval        30
contact_groups            boss
}

最后，编辑nagios.cfg文件
#vi /usr/local/nagios/etc/nagios.cfg
添加：
cfg_file=/usr/local/nagios/etc/objects/escalations.cfg

检查nagios配置文件是否正确：
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

重启nagios服务：
service nagios restart
【说明】报警从第4次起之后都是每隔30分钟发一次报警 ，发给admins 组，到第10次之后 admins组和boss 组都能收到报警 时间一各自配置文件为准，本例为30分钟
```

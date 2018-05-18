
## Nagios BPI（Business Process Intelligence）    

&emsp;&emsp;Nagios Business Process Intelligence is an advanced grouping tool that allows you to set more complex dependencies to determine groups states. Nagios BPI provides an interface to effectively view the ‘real’ state of the network. Rules for group states can be determined by the user, and parent-child relationships are easily identified when you need to ‘drill down’ on a problem. This tool can also be used in conjunction with a check plugin to allow for notifications through Nagios.  This document describes how to fully utilize the Nagios Business Process Intelligence (or BPI) add-on and incorporate checks into Nagios. 
&emsp;&emsp;Nagios Business Process Intelligence （BPI）是一种高级的分组工具，允许你设置更复杂的依赖关系来确定组状态。 Nagios BPI提供了一个界面来有效地查看网络的“真实”状态。 组状态的规则可以由用户确定。此工具也可以与检查插件结合使用，以通过Nagios进行通知。 本文档介绍如何充分利用Nagios业务流程智能（或BPI）附件，并将检查纳入Nagios。

```ruby
# cd /tmp
# wget https://github.com/NagiosEnterprises/nagiosbpi/archive/master.zip
# unzip master.zip 
# mv /tmp/nagiosbpi-master/nagiosbpi /usr/local/nagios/share/
# cd /usr/local/nagios/share/nagiosbpi
# mkdir tmp
# chmod +x set_bpi_perms.sh
# ./set_bpi_perms.sh 
## chown -R apache:nagios /usr/local/nagios/share/nagiosbpi/
# vim constants.conf
STATUSFILE=/usr/local/nagios/var/status.dat
OBJECTSFILE=/usr/local/nagios/var/objects.cache
CONFIGFILE=/usr/local/nagios/share/nagiosbpi/bpi.conf
CONFIGBACKUP=/usr/local/nagios/share/nagiosbpi/bpi.conf.backup
XMLOUTPUT=/usr/local/nagios/share/nagiosbpi/tmp/bpi.xml
```
http://192.168.5.113/nagios/nagiosbpi/
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-BPI1.png)
```js
define localServices1 {
                title=Local Services
                desc=Example BPI Group
                primary=1
                info=http://localhost
                members=localhost;Current Load;&, localhost;Current Users;&, localhost;HTTP;&, localhost;PING;|,
                warning_threshold=1
                critical_threshold=2
                priority=1

}

##################################
define localServices2 {
                title=More Local Services
                desc=Demo Group 2
                primary=1
                info=http://localhost/nagios
                members=$localServices1;&, localhost;Root Partition;|, localhost;SSH;&, localhost;Swap Usage;&, localhost;Total Processes;|,
                warning_threshold=3
                critical_threshold=4
                priority=2

}
【说明】每个members成员服务使用这种格式：localhost;Current Users;&,     主机名;检测的服务;&，   &表示非必要成员，|表示必要成员，使用这个符号服务名后面会显示**，各个成员服务名之间使用,分割

```
- A Basic BPI Group  
`This is a basic group with 5 members.The group has no thresholds set, and there are no essential members. Since there are still some members in an 'Ok' state, the group state is listed as 'Ok.'`  
【说明】一个基本的BPI组，这个基本的BPI组有5个成员，这个组没有阈值的设置，也没有“必要”的成员，只要这个组成员中有OK状态的，这个组的状态就是OK的   
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-BPI2.png) 

- A Group Using Thresholds   
`This  group has no essential members, but it has a warning threshold set at 3 problems, and a critical threshold set at 6 problems. Since the problem count of the group's members exceeds the warning threshold, the group state is 'Warning.'`  
【说明】一个使用了阈值的组，这个组没有“必要”的成员，但是设置上了超过3个问题就会有“warning”的阈值，超过6个问题就会有“critical”的阈值，
这个组的成员出现的问题数超过了“warning”设置的阈值（3个），所以这个组的状态就是“warning”  
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-BPI3.png) 

- A Group Using Essential Members   
`This group has 2 essential members defined, which are denoted with a '**' next to their state. If an essential member has a problem, the entire group will be in a problem state, even though the thresholds have not been exceeded, and there is only one problem.`  
【说明】这个组使用了“必要的”成员，这个组有2个必要的成员的定义，后面标记**的为必要的成员，如果一个必要的成员出现了问题，那么整个组将是问题状态，尽管没有超出阈值，也会存在一个问题


- Complex BPI Groups   
`The BPI groups determine state by looking down only one level. The BPI group will essentially look for the worst state trigger in the group, so if the warning threshold is exceeded for a group, but an essential member is “critical”, the group will still be “critical”. There is no limit to the number of sub groups that can be created, you can define as many levels in your dependency tree as you want.`  
【说明】这个BPI组只有一个水平的状态，这个BPI组将会找一个组内最糟糕的状态触发，所以如果组内的“warning”超出了阈值，但是一个必要的成员状态是“critical”（critical > warning），那么这个组的状态也是“critical”，子组的创建时没有限制的，你可以创建多个水平在你的依赖树状中  
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-BPI4.png) 

- Primary Groups   
`“Primary” BPI groups are seen from the top level of BPI page, while a non-primary group must have a visible parent group in order to be seen on the display. If a non-primary group is defined but never assigned as a member somewhere else, it will not be visible on the display.`  
【说明】“Primary”BPI组会在BPI的页面的最顶端看到，然而没有主组的必要要有一个依赖的“父组”，为了是能在页面中显示，如果一个“non-primary”组被定义了，但是没有委派任何的成员，那么这个组将不会再页面中显示
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-BPI5.png) 
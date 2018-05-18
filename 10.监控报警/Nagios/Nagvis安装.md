## Nagvis安装 

&emsp;&emsp;需要前提先安装NDOUtils，将Nagios监控信息存入MySQL

### 1、安装nagvis
```ruby
# yum install php-mbstring php-pdo graphviz rsync

# cd /nagios_soft/
# tar xvf nagvis-1.8.5.tar.gz
# mv nagvis-1.8.5 nagvis
# cd nagvis
# ./install.sh
[root@tomcat1 nagvis]# ./install.sh 
+------------------------------------------------------------------------------+
| Welcome to NagVis Installer 1.8.5                                            |
+------------------------------------------------------------------------------+
| This script is built to facilitate the NagVis installation and update        |
| procedure for you. The installer has been tested on the following systems:   |
| - Debian, since Etch (4.0)                                                   |
| - Ubuntu, since Hardy (8.04)                                                 |
| - SuSE Linux Enterprise Server 10 and 11                                     |
|                                                                              |
| Similar distributions to the ones mentioned above should work as well.       |
| That (hopefully) includes RedHat, Fedora, CentOS, OpenSuSE                   |
|                                                                              |
| If you experience any problems using these or other distributions, please    |
| report that to the NagVis team.                                              |
+------------------------------------------------------------------------------+
| Do you want to proceed? [y]: y
+------------------------------------------------------------------------------+
| Starting installation of NagVis 1.8.5                                        |
+------------------------------------------------------------------------------+
| OS  : CentOS release 6.5 (Final)                                             |
|                                                                              |
+--- Checking for tools -------------------------------------------------------+
| Using packet manager /bin/rpm                                          found |
|                                                                              |
+--- Checking paths -----------------------------------------------------------+
| Please enter the path to the nagios base directory [/usr/local/nagios]: 
|   nagios path /usr/local/nagios                                        found |
| Please enter the path to NagVis base [/usr/local/nagvis]: 
|                                                                              |
+--- Checking prerequisites ---------------------------------------------------+
| PHP 5.3                                                                found |
|   PHP Module: gd php                                                   found |
|   PHP Module: mbstring php                                             found |
|   PHP Module: gettext compiled_in                                      found |
|   PHP Module: session compiled_in                                      found |
|   PHP Module: xml php                                                  found |
|   PHP Module: pdo php                                                  found |
|   Apache mod_php                                                       found |
| Checking Backends. (Available: mklivestatus,ndo2db,ido2db)                   |
| Do you want to use backend mklivestatus? [y]: n
| Do you want to use backend ndo2db? [n]: y
| Do you want to use backend ido2db? [n]: n
|   /usr/local/nagios/bin/ndo2db-3x (ndo2db)                             found |
|   PHP Module: mysql php                                                found |
| WARNING: The Graphviz package was not found.                                 |
|          This may not be a problem if you installed it from source           |
|   Graphviz Module dot                                                MISSING |
|   Graphviz Module neato                                              MISSING |
|   Graphviz Module twopi                                              MISSING |
|   Graphviz Module circo                                              MISSING |
|   Graphviz Module fdp                                                MISSING |
| SQLite 3.6                                                             found |
|                                                                              |
+--- Trying to detect Apache settings -----------------------------------------+
| Please enter the web path to NagVis [/nagvis]: 
| Please enter the name of the web-server user [apache]: 
| Please enter the name of the web-server group [apache]: 
| create Apache config file [y]: y
|                                                                              |
+--- Checking for existing NagVis ---------------------------------------------+
|                                                                              |
+------------------------------------------------------------------------------+
| Summary                                                                      |
+------------------------------------------------------------------------------+
| NagVis home will be:           /usr/local/nagvis                             |
| Owner of NagVis files will be: apache                                        |
| Group of NagVis files will be: apache                                        |
| Path to Apache config dir is:  /etc/httpd/conf.d                             |
| Apache config will be created: yes                                           |
|                                                                              |
| Installation mode:             install                                       |
|                                                                              |
| Do you really want to continue? [y]: y
+------------------------------------------------------------------------------+
| Starting installation                                                        |
+------------------------------------------------------------------------------+
| Creating directory /usr/local/nagvis...                                done  |
| Creating directory /usr/local/nagvis/var...                            done  |
| Creating directory /usr/local/nagvis/var/tmpl/cache...                 done  |
| Creating directory /usr/local/nagvis/var/tmpl/compile...               done  |
| Creating directory /usr/local/nagvis/share/var...                      done  |
| Copying files to /usr/local/nagvis...                                  done  |
| Creating directory /usr/local/nagvis/etc/profiles...                   done  |
| Creating main configuration file...                                    done  |
| setting backend to ndomy_1                                             done  |
|   Adding webserver group to file_group...                              done  |
| Creating web configuration file...                                     done  |
| Setting permissions for web configuration file...                      done  |
|                                                                              |
|                                                                              |
|                                                                              |
+--- Setting permissions... ---------------------------------------------------+
| /usr/local/nagvis/etc/nagvis.ini.php-sample                            done  |
| /usr/local/nagvis/etc                                                  done  |
| /usr/local/nagvis/etc/maps                                             done  |
| /usr/local/nagvis/etc/maps/*                                           done  |
| /usr/local/nagvis/etc/geomap                                           done  |
| /usr/local/nagvis/etc/geomap/*                                         done  |
| /usr/local/nagvis/etc/profiles                                         done  |
| /usr/local/nagvis/share/userfiles/images/maps                          done  |
| /usr/local/nagvis/share/userfiles/images/maps/*                        done  |
| /usr/local/nagvis/share/userfiles/images/shapes                        done  |
| /usr/local/nagvis/share/userfiles/images/shapes/*                      done  |
| /usr/local/nagvis/var                                                  done  |
| /usr/local/nagvis/var/*                                                done  |
| /usr/local/nagvis/var/tmpl                                             done  |
| /usr/local/nagvis/var/tmpl/cache                                       done  |
| /usr/local/nagvis/var/tmpl/compile                                     done  |
| /usr/local/nagvis/share/var                                            done  |
|                                                                              |
+------------------------------------------------------------------------------+
| Installation complete                                                        |
|                                                                              |
| You can safely remove this source directory.                                 |
|                                                                              |
| For later update/upgrade you may use this command to have a faster update:   |
| ./install.sh -n /usr/local/nagios -p /usr/local/nagvis -b ndo2db -u apache -g apache -w /etc/httpd/conf.d -a y
|                                                                              |
| What to do next?                                                             |
| - Read the documentation                                                     |
| - Maybe you want to edit the main configuration file?                        |
|   Its location is: /usr/local/nagvis/etc/nagvis.ini.php                      |
| - Configure NagVis via browser                                               |
|   <http://localhost/nagvis/config.php>                                       |
| - Initial admin credentials:                                                 |
|     Username: admin                                                          |
|     Password: admin                                                          |
+------------------------------------------------------------------------------+
```
### 2、修改nagvis配置文件
```ruby
# vim /usr/local/nagvis/etc/nagvis.ini.php
ndo2db MySQL backend (ndomy)
The ndo2db MySQL backend, in short ndomy backend, is used to fetch Nagios information like status and configuration data via a MySQL database. The Nagios addon called ndoutils stores all information which are present in a running Nagios in a MySQL database. This database is being queried by the NagVis ndomy backend.
You can use the following parameters to configure an ndomy backend:

dbhost="localhost"
dbport=3306
dbname="nagios"
dbuser="root"
dbpass="wangzongyu"
dbprefix="nagios_"
dbinstancename="default"
maxtimewithoutupdate=180
htmlcgi="/nagios/cgi-bin"

# vim /etc/httpd/conf/httpd.conf
添加：ServerName localhost:80
# service httpd restart
```
### 3、添加一个MAP：
`Options`  -> `Manage Maps` -> `Create Map中填写ID(tradeease)`
`Open` -> `选择上面创建的Map`

添加主机、添加相关服务：  
`都是在上面的MAP下操作`
`Edit Map` -> `Add Icon` ->

如果要对主机或者服务图标操作，需要先Unlock和lock，右键图标操作即可

`图片格式都是png`   
`Logo`:/usr/local/nagvis/share/frontend/nagvis-js/images/internal  
`背景图片路径`：/usr/local/nagvis/share/userfiles/images/maps  
`服务器图片路径`：/usr/local/nagvis/share/userfiles/images/iconsets

`Edit Map` -> `Map Options`
![](https://github.com/ZongYuWang/image/blob/master/Nagios/Nagios-Nagvis1.png)

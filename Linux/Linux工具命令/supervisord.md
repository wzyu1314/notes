supervisord
===

https://blog.csdn.net/weixin_40680612/article/details/124422102

Supervisor是用Python开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启。它是通过fork/exec的方式把这些被管理的进程当作supervisor的子进程来启动，这样只要在supervisor的配置文件中，把要管理的进程的可执行文件的路径写进去即可。也实现当子进程挂掉的时候，父进程可以准确获取子进程挂掉的信息的，可以选择是否自己启动和报警。supervisor还提供了一个功能，可以为supervisord或者每个子进程，设置一个非root的user，这个user就可以管理它对应的进

配置后台服务/常驻进程的进程管家工具。
supervisord的出现，可以用来管理后台运行的程序。通过supervisorctl客户端来控制supervisord守护进程服务，真正进行进程监听的是supervisorctl客户端，而运行supervisor服务时是需要制定相应的supervisor配置文件的。

## 安装

```shell
# 安装 supervisord
apt-get install supervisor
```

```
启动
systemctl start supervisord.service
停止
systemctl start supervisord.service
重启
systemctl restart supervisord.service
查看状态
systemctl status supervisord.service
```

## 使用

Supervisord工具的整个使用流程：
1. 首先通过echo_supervisord_conf 生成配置文件模板
2. 然后你根据自己的需求进行修改，接着就使用相应的命令来使用supervisorctl客户端
3. 而supervisorctl客户端会将对应的信息传递给supervisord守护进程服务，让supervisord守护进程服务进行进程守护。
## 实例

生成配置文件 `/etc/supervisord.conf`

```shell
[program:app]
command=/usr/bin/gunicorn -w 1 wsgiapp:application
directory=/srv/www
user=www-data
```

supervisord: 启动 supervisor 服务

```shell
supervisorctl start app
supervisorctl stop app
supervisorctl reload # 修改/添加配置文件需要执行这个
supervisorctl status
webserver                        RUNNING   pid 1120, uptime 0:08:07
supervisorctl status 查看进程运行状态
supervisorctl start 进程名 启动进程
supervisorctl stop 进程名 关闭进程
supervisorctl restart 进程名 重启进程
supervisorctl update 重新载入配置文件
supervisorctl shutdown 关闭supervisord
supervisorctl clear 进程名 清空进程日志
supervisorctl 进入到交互模式下。使用help查看所有命令。
start stop restart + all 表示启动，关闭，重启所有进程进程名 清空进程日志 
supervisorctl 进入到交互模式下。使用help查看所有命令。 
start stop restart + all 表示启动，关闭，重启所有进程 
```

启动supervisor程序

```shell
supervisord -c /home/nianshi/supervisor/conf/supervisord.conf
```

## 下载地址

https://pypi.python.org/pypi/meld3  
https://pypi.python.org/pypi/supervisor  


## 配置文件

一般apt安装后配置文件默认位置是/etc/supervisor/supervisord.conf。其中注释是以分号开头
```shell
#指定了socket file的位置
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid
 
 #用于启动一个含有前端的服务，可以从Web页面中管理服务。其中，port用于设置访问地址，username和password用于设置授权认证。
;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码
 
 # 管理服务本身的配置
[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200
 
 
[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord
 
; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程
 # 对事件进行的管理
;[eventlistener:theeventlistenername]

#对任务组的管理 ,包含其它配置文件
;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

[include]
files = supervisord.d/*.ini    ;可以指定一个或多个以.ini结束的配置文件
```

```shell
command 要执行的命令
priority 优先级
numprocs 启动几个进程
autostart supervisor启动的时候是否随着同时启动
autorestart 当程序over的时候，这个program会自动重启，一定要选上
```


```shell
配置监控应用
cd /etc/supervisord.d
vim frpserver.conf
[program:frpServer]                     ; 程序名称，可以通过ctl指定名称进行控制
#directory = /home/kangaroo/build/CIServer             ; 程序的启动目录
command = /root/frp/frps -c /root/frp/frps.ini
#  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true                        ; 在 supervisord 启动的时候也自动启动
startsecs = 20                          ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true                      ; 程序异常退出后自动重启
startretries = 3                         ; 启动失败自动重试次数，默认是 3
user = root                              ; 用哪个用户启动
redirect_stderr = true                   ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile = /home/supervisor/log/frps.log
stdout_logfile_maxbytes = 20MB           ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20              ; stdout 日志文件备份数

tomcat配置：

cd /etc/supervisord.d
vim frpserver.conf
[program:tomcat]
command=/apache-tomcat-8.5.55/bin/catalina.sh run
environment=JAVA_HOME="/java/jdk1.8.0_191/",JAVA_BIN="/java/jdk1.8.0_191/bin"
directory=/apache-tomcat-8.5.55/
autostart = true
autorestart=true
redirect_stderr=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=20MB

nginx配置

cd /etc/supervisord.d
vim frpserver.conf
[program:nginxServer]                       ; 程序名称，可以通过ctl指定名称进行控制
#directory = /home/kangaroo/build/CIServer             ; 程序的启动目录
command = /usr/sbin/nginx -g 'daemon off;'
#  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true                    ; 在 supervisord 启动的时候也自动启动
startsecs = 20                      ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true                  ; 程序异常退出后自动重启
startretries = 3                    ; 启动失败自动重试次数，默认是 3
user = root                         ; 用哪个用户启动
redirect_stderr = true              ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile = /home/supervisor/log/nginx.log
stdout_logfile_maxbytes = 20MB       ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20          ; stdout 日志文件备份数
```


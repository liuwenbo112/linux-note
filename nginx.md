Nginx

https://www.cnblogs.com/pyyu/p/9468680.html

#### 1、安装nginx步骤

	1.yum解决编译nginx所需的依赖包，之后你的nginx就不会报错了
	
	yum install gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel openssl openssl-devel -y
	
	2.安装配置nginx软件，下载源代码
	
		wget -c https://nginx.org/download/nginx-1.12.0.tar.gz
		
	3.解压缩源码，编译且安装
		tar -zxvf nginx-1.12.0.tar.gz 
		切换源码目录
		./configure --prefix=/opt/nginx112/
		make && make install 
		
	4.进入nginx的工作目录
	cd /opt/ngin112/
	
	5.查看gninx的工作目录
	[root@localhost nginx112]# ls
	conf  配置文件目录
	html  网页根目录，你的index.html就放在这里，然后通过域名访问  pythonav.cn/index.html     html/index.html 
	logs	日志
	sbin	存放nginx可执行命令的
	
	6.定制自己的nginx网站
	修改/opt/nginx112/html/index.html  这是nginx网页根文件，清空内容写入自己的html标签
	
	7.启动nginx服务器
	/opt/nginx112/sbin/nginx    直接回车执行 
	
	8.检查nginx服务端口
	ps -ef|grep nginx 
	
	9.通过windows访问nginx web服务
	浏览器 访问http://192.168.13.79(根据自己的IP自行改变)

2. nginx.conf主配置文件学习（/opt/nginx112/conf/nginx.conf）	

    	```
     worker_processes  1;   nginx工作进程数，根据cpu的核数定义
        	events {
        	    worker_connections  1024;    #事件连接数
        	}
        	#http区域块，定义nginx的核心web功能
        	http {
        	    include(关键字)       mime.types(可修改的值);
        	    default_type  application/octet-stream;
        	#定义日志格式
        	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        	                  '$status $body_bytes_sent "$http_referer" '
        	                  '"$http_user_agent" "$http_x_forwarded_for"';
        	#开启访问日志功能的参数		  
        	access_log  logs/access.log  main;
        	sendfile        on;
        	#tcp_nopush     on;
        	#keepalive_timeout  0;
        	#保持长连接
        	keepalive_timeout  65;
        	#支持图片 gif等等压缩，减少网络带宽
        	gzip  on;
        	
        	#这个server标签 控制着nginx的虚拟主机(web站点)
        	server {
        		# 定义nginx的入口端口是80端口
        	    listen       80;
     		# 填写域名，没有域名就写ip地址
        	    server_name  www.s15rihan.com;
     		# 定义编码格式
        	    charset utf-8;
     		# location定义网页的访问url
        		#就代表 用户的请求 是  192.168.13.79/
        	    location / {
        			#root参数定义网页根目录
        	        root   html;
        			#定义网页的首页文件，的名字的
        	        index  index.html index.htm;
        	    }
        		#定义错误页面，客户端的错误，就会返回40x系列错误码
        	    error_page  404  403 401 400            /404.html;
        		#500系列错误代表后端代码出错
        	    error_page   500 502 503 504  /50x.html;
        	}
        	#在另一个server{}的外面，写入新的虚拟主机2
        	server{
        		listen 80;
        		server_name  www.s15oumei.com;
        		location /  {
        		root  /opt/myserver/oumei;		#定义虚拟主机的网页根目录
        		index  index.html;
        		}
        	}
    }
    ```
    
    
    
    3.准备两个虚拟主机的网页根目录内容
    
    ```
    [root@localhost myserver]# tree /opt/myserver/
    		/opt/myserver/
    		├── oumei
    		│   └── index.html		写入自己的内容
    		└── rihan
    			└── index.html		写入自己的内容 
    ```
    
    ​		
    4.修改windows本地的测试域名  C:\Windows\System32\drivers\etc\hosts文件
    写入如下内容
    
        192.168.13.79 www.s15rihan.com  
        
        192.168.13.79 www.s15oumei.com  
        
        因为我们没有www.s15oumei.com 也没有  www.s15rihan.com ，因此要在本地搞一个测试域名，
        不想改dns的话，就去阿里云去买一个域名~~~~~~~~~~~~~~~~~~~~~~
    
    5.然后在浏览器测试访问 两个不同的 web站点
    
    www.s15rihan.com  
    
    www.s15oumei.com   
    

#### nginx的访问日志功能

    1.开启nginx.conf中的日志参数
    
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    #开启访问日志功能的参数		  
    access_log  logs/access.log  main;
    
    2.检查access.log的日志信息
    
    tail -f  access.log 

####     nginx的拒绝访问功能


    1.在nginx.conf中，添加参数
    在server{}虚拟主机标签中，找到location 然后添加参数
    #当赵一宁访问  192.168.13.79/  的时候 
        location / {
    		#拒绝参数是 deny 
    		#deny 写你想拒绝的IP地址
    		#deny还支持拒绝一整个网段，例如拒绝192.168.13.0/24 则192.168.13.这一网段开头的均不能访问
            deny  192.168.13.33;
            root   /opt/myserver/rihan;
            index  index.html;
        }


#### nginx的错误页面优化

```
1.修改nginx.conf 中的配置参数
这个s1540x.html存在 虚拟主机定义的网页根目录下（/opt/nginx112/html/下定义s1540x.html）
  error_page  404              /s1540x.html;
```

#### nginx的反向代理功能(自带了反向代理的功能，天生的二道贩子)


	1.实验环境准备
	准备2个服务器，都安装好nginx软件
	nginx1		192.168.13.79   作为web服务器 (理解为火车票售票点)
	
	nginx2		192.168.13.24	作为反向代理服务器		(黄牛)
	
	用户   通过浏览器去访问   黄牛 （代理）(例如)
	浏览器 访问  192.168.13.24    >		192.168.13.79
	
	2.在反向代理服务器中的nginx.conf文件夹添加配置
	server {
	        listen       80;
	        server_name  localhost;# 这里举例是在本机上作为反向代理服务器
	        location / {
	        # 添加代理 没有域名写ip地址
	        proxy_pass  http://slave_pools;
	            #root   html;
	            #index  index.html index.htm;
	        }
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	            
	3.在web服务器中填写
		server {
	        listen       80;
	        server_name  slave_pools;
	        location / {
	            #root   html;
	            #index  index.html index.htm;
	        }
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;

#### nginx负载均衡

https://www.cnblogs.com/pyyu/p/10004583.html

https://www.cnblogs.com/pyyu/p/10004681.html

```
集群的概念：一堆服务器做一件事

1.实验准备
准备三台计算机 

nginx1  	192.168.13.121 作为nginx负载均衡器，只要我访问这个负载均衡器，查看页面的结果，到底是来自于

nginx2  	192.168.13.24	web服务，提供一个页面		

nginx3 		192.168.13.79  web服务，提供一个页面 



2.先配置两个nginx  web页面  
	192.168.13.24  准备一个   index.html  写入  你好，我是192.168.13.24机器
	192.168.13.79	准备一个	index.html 写入		老了老弟，我是192.168.13.79
	

​```
然后启动两个nginx web 服务
​```

3.准备一个nginx负载均衡器  192.168.13.121机器上，修改nginx.conf 
在nginx.conf > http 区域中
#写入如下内容 
			#定义一个负载均衡池，负载均衡的算法有
			#调度算法   　　 概述
			#轮询    　　　　按时间顺序逐一分配到不同的后端服务器(默认)
			#weight  　　   加权轮询,weight值越大,分配到的访问几率越高
			#ip_hash   　　 每个请求按访问IP的hash结果分配,这样来自同一IP的固定访问一个后端服务器
			#url_hash   　  按照访问URL的hash结果来分配请求,是每个URL定向到同一个后端服务器
			#least_conn    最少链接数,那个机器链接数少就分发
			#1.轮询(不做配置，默认轮询)

            #2.weight权重(优先级)

            #3.ip_hash配置，根据客户端ip哈希分配，不能和weight一起用
           
            upstream s15webserver  {
            ip_hash;
            server 192.168.13.79 ;
            server 192.168.13.24 ;
            }

4、然后在虚拟主机中添加 反向代理配置，将用户的请求，直接转发给 负载均衡池中的服务器
	在nginx.conf > http 区域 >  server区域  > location配置中  添加proxy_pass
	
    server {
            listen       80;
            #当我的请求来自于 192.168.13.121时，走这>个虚拟主机
            server_name  192.168.13.121;
            #charset koi8-r;
            #access_log  logs/host.access.log  main;
            #核心配置，就在这，一条proxy_psss参数即可，s15webserver与第三步中upstream（负载均衡池）的名字保持一致
            location / {
              proxy_pass http://s15webserver;
                #root   html;
                #index  index.html index.htm;
            }
     }

4.启动负载均衡器的 nginx服务 

5.在客户端windows中测试访问，负载均衡器  192.168.13.121 ，查看请求分发的结果 
```

### [vue+uwsgi+nginx部署路飞学城]

```
(https://www.cnblogs.com/pyyu/p/10160874.html)
https://www.cnblogs.com/pyyu/p/9481344.html
```

#### 上线部署需要的环境

```
python3 

uwsgi    wsgi(web服务网关接口，就是一个实现了python web应用的协议)

virtualenvwrapper

路飞的代码

vue的代码

nginx (一个是nginx对静态文件处理的优秀性能，一个是nginx的反向代理功能，以及nginx的默认80端口，访问nginx的80端口，就能反向代理到应用的8000端口)

mysql 

redis   购物车订单信息

supervisor 进程管理工具 
```

#### 1、新建一个虚拟环境  s15vuedrf 

```
mkvirtualenv  s15vuedrf 
```

#### 2、准备前后端代码

```
wget https://files.cnblogs.com/files/pyyu/luffy_boy.zip 
wget https://files.cnblogs.com/files/pyyu/07-luffy_project_01.zip
如果代码在本地，传到服务器 使用 lrzsz 和xftp工具
```

#### 3、解压缩代码

```
.zip文件用unzip解压
unzip luffy_boy.zip 
unzip 07-luffy_project_01.zip
```

#### 4、先从前端vue搞起

```
要在服务器上，编译打包vue项目，必须得有node环境
下载node二进制包，此包已经包含node，不需要再编译
wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz
解压缩
tar -zxvf node-v8.6.0-linux-x64.tar.gz
进入node文件夹
[root@web02 opt]# cd node-v8.6.0-linux-x64/
[root@web02 node-v8.6.0-linux-x64]# ls
bin  CHANGELOG.md  etc  include  lib  LICENSE  README.md  share
[root@web02 node-v8.6.0-linux-x64]# ls bin
node  npm  npx
[root@web02 node-v8.6.0-linux-x64]# ./bin/node -v
v8.6.0
[root@web02 node-v8.6.0-linux-x64]# ./bin/npm -v
5.3.0
```

#### 5、node环境有了，安装node模块，以及打包node项目

```
cd /opt/s15vuedrf/07-luffy_project_01
输入npm install  
	npm run build  
这两条都正确配置之后，就会生成一个 dist 静态文件目录，整个项目的前端内容和index.html都在这里了


准备编译打包vue项目，替换配置文件所有地址，改为服务器地址（原来api.js里面写的是127.0.0.1）
sed -i 's/127.0.0.1/192.168.119.12/g' /opt/07-luffy_project_01/src/restful/api.js

5.等待nginx加载这个 dist文件夹
```

#### 6、部署后端代码所需的环境

	1.激活虚拟环境
			workon s15vuedrf 
	2.通过一条命令，导出本地的所有软件包依赖 
			pip3 freeze >  requirements.txt 
	3.将这个requirements.txt 传至到服务器，在服务器的新虚拟环境中，安装这个文件，就能安装所有的软件包了
			pip3 install -r  requirements.txt   
		这个文件内容如下：项目所需的软件包都在这里了
			[root@web02 opt]# cat requirements.txt
			certifi==2018.11.29
			chardet==3.0.4
			crypto==1.4.1
			Django==2.1.4
			django-redis==4.10.0
			django-rest-framework==0.1.0
			djangorestframework==3.9.0
			idna==2.8
			Naked==0.1.31
			pycrypto==2.6.1
			pytz==2018.7
			PyYAML==3.13
			redis==3.0.1
			requests==2.21.0
			shellescape==3.4.1
			urllib3==1.24.1
			uWSGI==2.0.17.1
		
	4.准备uwsgi 支持高并发的启动python项目(注意uwsgi不支持静态文件的解析，必须用nginx去处理静态文件)
		1.安装uwsgi 
		 pip3 install -i https://pypi.douban.com/simple uwsgi
	
		2.学习uwsgi的使用方法
		
			通过uwsgi启动一个python web文件
			
			自定义一个s15testuwsgi.py文件，内容如下：
			
	            def application(env, start_response):
	                start_response('200 OK', [('Content-Type','text/html')])
	                return [b"Hello World"] # python3
	            
			uwsgi --http :8000 --wsgi-file   s15testuwsgi.py
					--http 指定http协议 
					--wsgi-file  指定一个python文件
			
		3.通过uwsgi启动django项目，并且支持热加载项目，不重启项目，自动生效 新的 后端代码
			
			uwsgi --http  :8000 --module s15drf.wsgi    --py-autoreload=1
		
			--module 指定找到django项目的wsgi.py文件
					
	5.使用uwsgi的配置文件，启动项目
			1.新建创建一个uwsgi.ini配置文件，写入参数信息在
				touch uwsgi.ini 
									
					[uwsgi]
					# Django-related settings
					# the base directory (full path)
					#指定项目的绝对路径的第一层路径！！！！！！！！！！！！！！！！！！！！！！！！
					chdir           = /opt/s15drfvue/luffy_boy
	
					# Django's wsgi file
					#  指定项目的 wsgi.py文件！！！！！！！！！！！！
					#  写入相对路径即可，这个参数是以  chdir参数为相对路径
					module          = luffy_boy.wsgi
	
					# the virtualenv (full path)
					# 写入虚拟环境解释器的 绝对路径！！！！！！
					home            = /root/Envs/s15vuedrf
					# process-related settings
					# master
					master          = true
					# maximum number of worker processes
					#指定uwsgi启动的进程个数				
					processes       = 1
	
					#这个参数及其重要！！！！！！
					#这个参数及其重要！！！！！！
					#这个参数及其重要！！！！！！
					#这个参数及其重要！！！！！！
					# the socket (use the full path to be safe
					#socket指的是，uwsgi启动一个socket连接，当你使用nginx+uwsgi的时候，使用socket参数
					socket          = 0.0.0.0:8000
	
					#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
					#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
					#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
					#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
					#这个参数是uwsgi启动一个http连接，当你不用nginx只用uwsgi的时候，使用这个参数
					#http  =  0.0.0.0:8000
	
					# ... with appropriate permissions - may be needed
					# chmod-socket    = 664
					# clear environment on exit
					vacuum          = true
					# 后台运行参数
					daemonize=yes
					
	6.使用uwsgi配置文件启动项目
	uwsgi --ini  uwsgi.ini 

##### supervisor进程管理工具

	1.将linux进程运行在后台的方法有哪些
	   第一个，命令后面加上 &  符号
	   python  manage.py  runserver & 
	   第二个 使用nohup命令	 
	   第三个使用进程管理工具
		
	2.安装supervisor，使用python2的包管理工具 easy_install ，注意，此时要退出虚拟环境！！！！
	如果没有命令，使用以下命令，安装
	yum install python-setuptools
	easy_install supervisor
	
	3.通过命令，生成一个配置文件，这个文件就是写入你要管理的进程任务
	echo_supervisord_conf > /etc/supervisor.conf
	
	4.编辑这个配置文件，写入操作  django项目的 命令 
	vim /etc/supervisor.conf  
	直接到最底行，写入以下配置
	[program:s15luffy](s15luffy是自定义的)
	command=/root/Envs/s15vuedrf/bin/uwsgi  --ini  /opt/s15drfvue/uwsgi.ini
	
	5.启动supervisord服务端，指定配置文件启动
	supervisord -c  /etc/supervisor.conf
	
	6.通过supervisorctl管理任务
	supervisorctl -c /etc/supervisor.conf 
	
	7.supervisor管理django进程的命令如下
	supervisorctl直接输入命令会进入交互式的操作界面
	
	> stop s15luffy 
	> start s15luffy 
	> status s15luffy 
	
	8.启动luffy的后端代码				

##### 配置nginx步骤如下

```
1.编译安装nginx
2.nginx.conf配置如下

#第一个server虚拟主机是为了找到vue的dist文件， 找到路飞的index.html
server {
        listen       80;
        server_name  192.168.13.79;
		#当请求来自于 192.168.13.79/的时候，直接进入以下location，然后找到vue的dist/index.html 
        location / {
            root   /opt/s15vuedrf/07-luffy_project_01/dist;
            index  index.html;
        }
}
#由于vue发送的接口数据地址是 192.168.13.79:8000  我们还得再准备一个入口server
server {
	listen 8000;:
	server_name  192.168.13.79;
	#当接收到接口数据时，请求url是 192.168.13.79:8000 就进入如下location
    location /  {
        #这里是nginx将请求转发给  uwsgi启动的 9000端口
        uwsgi_pass  192.168.13.79:9000;
        # include  就是一个“引入的作用”，就是将外部一个文件的参数，导入到当前的nginx.conf中生效
        include /opt/nginx112/conf/uwsgi_params;
    }
   }
   
3.启动nginx 
./sbin/nginx  直接启动 
此时可以访问 192.168.13.79  查看页面结果



启动路飞项目，这个项目用的是sqllite，因此安装mysql自行选择

redis必须安装好，存放购物车的数据
```


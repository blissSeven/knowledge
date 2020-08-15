# resin 配置
* http://caucho.com/resin-4.0/admin/starting-resin-install.xtp
* ./configure
* sudo resinctl start  127.0.0.1:8080
* sudo resinctl stop

Deployment Directories
When deploying, it's a good idea to create a bit of structure to make Resin and website upgrades easier and more maintainable.

Create a user to run Resin (e.g. resin or another non-root user)
Link /usr/local/share/resin to the current Resin directory. This is $RESIN_HOME.
Create a deployment root, e.g. /var/resin, owned by the resin user. This is $RESIN_ROOT.
Put the modified resin.xml in /etc/resin/resin.conf
Put the site documents in /var/resin/webapps/ROOT.
Put any .war files in /var/resin/webapps.
Put any virtual hosts in /var/resin/hosts/www.foo.com.
Refer to virtual hosts for more information.
Output logs will appear in /var/resin/log.
Create a startup script and configure the server to start it when the machine reboots.

cp spittr.war  /usr/local/share/resin-4.0.65
sudo resinctl deploy spittr.war


    <cluster id="web2">
    <server id="web2" port="6802">
     <http id="" port="8082"/>
    </server>
    
    <host id="" root-directory=".">
      <!-- <web-app id="/" root-directory="/home/bliss/resintasks/app1/spittr" redeploy-mode="manual"/> -->
       <web-app id="/" root-directory="/home/bliss/resintasks/app2/spittr"/>
        <!-- <web-app id="/" root-directory="webapps/ROOT"/> -->
    </host>
  </cluster>


resinctl  start  -server  web1


nginx -s stop
nginx -s reload

  sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev //安装依赖库

配置文件	/usr/local/nginx/conf/nginx.conf
程序文件	/usr/local/nginx/sbin/nginx
日志	/var/log/nginx
默认虚拟主机	/var/www/


 sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx



    #gzip  on;
    upstream resin{
        server localhost:8081;
        server localhost:8082;
        server localhost:8080;
    }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_pass http://resin;
            root   html;
            index  index.html index.htm;
        }
        
作者：XDiong
链接：https://www.jianshu.com/p/cfed9b17a18b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
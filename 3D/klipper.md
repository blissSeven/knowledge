# klipper
## klipper安装
https://github.com/th33xitus/kiauh.git  
安装klipper时注意目录权限  
/etc/nginx的目录权限  
/etc/nginx/sites-available  
/etc/nginx/sites-enabled  
Moonraker  
 192.168.242.28:7125  
 单机会修改本机的配置,会导致开启异常等情况,寻求docker实现
## klipper 容器化 
https://github.com/dimalo/klipper-web-control-docker 
串口设备的映射问题   
当没有串口设备时,可以先注释掉klipper 的Device项 
fluidd启动问题  
参考下https://github.com/dimalo/klipper-web-control-docker/issues/29,将fluidd的镜像改下即可  

## klipper 文档
nsenter -t <pid> -n ip address
 

# vsftpd
安装docker（略）
启动vsftpd服务：
注意：ftp支持主动和被动模式，主动模式需要公司防火墙开放大范围的端口；
而被动模式根据并发在服务器开放小范围端口就行21100-21104，所以我选择使用被动模式
docker run -d --name vsftpd \
--restart=always \
--cpuset-cpus="0,1" \
-m 1g --memory-swap -1 \
-p 21:21 -p 21100-21104:21100-21104 \
-e FTP_USER=dev -e FTP_PASS=dev \
-e PASV_ADDRESS=47.107.43.3 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21104 \
-v /data/ftp/file:/home/vsftpd \
-v /data/ftp/conf/vsftpd.conf:/etc/vsftpd/vsftpd.conf \
fauria/vsftpd

我解释一下我的启动命令：
-d 						              #后台启动
--restart=always		        #开机启动
--cpuset-cpus="0,1"		      #cpu只是用0，1核
-m 1g --memory-swap -1 	    #内存1g 不限制交换内存使用，反正我系统swap为0
-p 21:21				            #ftp监听端口
-p 21100-21104:21100-21104 			  #pasv端口，就是被动端口范围，配合配置文件进行设定就是这两个参数的范围：PASV_MIN_PORT=21100  PASV_MAX_PORT=21104
-e FTP_USER=dev -e FTP_PASS=dev		#用户/密码
-e PASV_ADDRESS=47.107.43.3			  #这个参数一定要设为公网，因为我的ftp服务器做了nat映射！！！
-v /data/ftp/file:/home/vsftpd		#映射用户家目录
-v /data/ftp/conf/vsftpd.conf:/etc/vsftpd/vsftpd.conf		#映射配置文件
fauria/vsftpd			#镜像名称

镜像来自docker官网：hub.docker.com
可以查看里面的说明，有启动方法介绍

添加用户方法：

docker exec -i -t vsftpd bash
mkdir /home/vsftpd/dev
echo -e "dev\dev" >> /etc/vsftpd/virtual_users.txt
/usr/bin/db_load -T -t hash -f /etc/vsftpd/virtual_users.txt /etc/vsftpd/virtual_users.db
exit
docker restart vsftpd

vsftpd配置文件

# Run in the foreground to keep the container running:
background=NO

# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
anonymous_enable=NO

# Uncomment this to allow local users to log in.
local_enable=YES

## Enable virtual users
guest_enable=YES

## Virtual users will use the same permissions as anonymous
virtual_use_local_privs=YES

# Uncomment this to enable any form of FTP write command.
write_enable=YES

## PAM file name
pam_service_name=vsftpd_virtual

## Home Directory for virtual users
user_sub_token=$USER
local_root=/home/vsftpd/$USER

# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
chroot_local_user=YES           #所有用户都被限制在其主目录下，可以配合chroot_list_enable=YES 指定用户列表chroot_list_file 不受限制

# Workaround chroot check.
# See https://www.benscobie.com/fixing-500-oops-vsftpd-refusing-to-run-with-writable-root-inside-chroot/
# and http://serverfault.com/questions/362619/why-is-the-chroot-local-user-of-vsftpd-insecure
#allow_writeable_chroot=YES
allow_writeable_chroot=YES      #允许主目录有写的权限

## Hide ids from user
hide_ids=YES

## Enable logging
xferlog_enable=YES
xferlog_file=/var/log/vsftpd/vsftpd.log

## Enable active mode
port_enable=YES
connect_from_port_20=YES
ftp_data_port=8041

## Disable seccomp filter sanboxing
pasv_address=47.107.43.3
pasv_max_port=21104
pasv_min_port=21100
pasv_addr_resolve=NO
pasv_enable=YES
file_open_mode=0666
local_umask=077
xferlog_std_format=NO

filezilla客户端链接：

搭建ubantu多人服务器教程
1.	配置了SSH服务
具体方法：
一、打开终端，输入以下命令安装OpenSSH服务：

sudo apt-get install openssh-server

二、步骤一的作用是在当前系统增加SSH服务，接下去可以用命令检查一下服务的状态。

sudo service ssh status 

如果需要对SSH服务修改设置，可以用字处理工具编辑其配置文件，位于“ /etc/ssh/sshd_config”，比如用vim修改的命令就是：

sudo nano /etc/ssh/sshd_config

三、初次安装后，其实SSH并没有自动运行，可以用server命令来启动，例如：

sudo service ssh restart

四、安装完成之后，就可以在客户端用系统用户来远程登录了。

例如在Windows系统下可以用Putty之类软件，在Linux中则可以使用openssh-client，可以用命令来安装：

sudo apt-get install openssh-client。

2.  配置Samba服务
1.先要安装Samba
sudo apt-get install samba openssh-server
2.编译Samba配置文件
sudo vim /etc/samba/smb.conf
找到[homes]项，此项默认是注释掉的，取消其注释，然后修改其具体内容，修改成如下：
[homes]
   comment = Home Directories
   browseable = yes
# By default, the home directories are exported read-only. Change the
# next parameter to 'no' if you want to be able to write to them.
   read only = no
# File creation mask is set to 0700 for security reasons. If you want to
# create files with group=rw permissions, set next parameter to 0775.
   create mask = 0755 #建议将权限修改成0755，这样其它用户只是不能修改
# Directory creation mask is set to 0700 for security reasons. If you want to
# create dirs. with group=rw permissions, set next parameter to 0775.
   directory mask = 0755
# By default, \\server\username shares can be connected to by anyone
# with access to the samba server. Un-comment the following parameter
# to make sure that only "username" can connect to \\server\username
# The following parameter makes sure that only "username" can connect
#
# This might need tweaking when using external authentication schemes
   valid users = %S #本行需要取消注释，%s  是指登陆用户可以访问 
如上修改完成后wq保存退出！
3. 重启samba服务：
sudo service restart smbd
sudo service restart nmbd
这里重启我用的是
sudo /etc/init.d/smbd restart
sudo /etc/init.d/nmbd restart
4. 增加一个现有用户的对应samba帐号：
如我已经有一个用户叫liuyan，现在给liuyan开通samba帐号：
sudo smbpasswd -a liuyan
根据提示输入两次密码即可（这个密码就是系统登陆账户liuyan的密码）
5.现在可以测试了，在Window下输入samba地址尝试登录：
\\10.0.0.2\reddy
6.此时windows应该会弹出窗口要求输入用户名和密码了，输入吧。Enjoy！如果没有提示输入密码，请重启Windows系统


3.	安装Xrdp插件：
下载地址：http://www.c-nergy.be/products.html

解压：unzip xrdp-installer-1.4.7.zip （根据文件名更改）
修改权限：chmod +x  ~/Downloads/xrdp-installer-1.4.7.sh（根据文件名更改）
安装：./xrdp-installer-1.4.7.sh


4.	常见问题
4.1连接黑屏
这个问题，主要是当你的本机没有注销的话，远程桌面就会黑屏，最佳解决策略就是退出本地登录，也就是注销登录，这个方法一定没问题。与windows那种完美的远程控制不同，在ubuntu中，本地登录和远程登陆是隔离开的，远程登录了不注销，那么本地就会黑屏，反过来本地登陆了不注销，远程就会黑屏。所谓注销就是logout，应该都懂，就是和关机、重启放在一起的那个选项。

或者使用网上的一些解决方案，但是这个放在在Ubuntu 22中会导致闪退。即，编辑 /etc/xrdp/startwm.sh 文件：

1. 打开文件
sudo vim /etc/xrdp/startwm.sh
2. 添加配置
unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
3. 重启xrdp服务
sudo systemctl restart xrdp.service

4.2桌面优化
注意，一定要先修改下面配置文件，再远程连接，否则会黑屏，这个时候需要重启。
反正记住一句话，重启后不在本地登录，那么远程必不黑屏！
如果不做任何配置，启动之后的桌面是非常别扭的，因为是Gnome的原始桌面，没有左侧的任务栏，窗口也没有最小化按钮，等等一些列问题。解决方案也很简单：

1. 添加配置文件
vim ~/.xsessionrc

添加：

export GNOME_SHELL_SESSION_MODE=ubuntu
export XDG_CURRENT_DESKTOP=ubuntu:GNOME
export XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
2. 重启xrdp服务
sudo systemctl restart xrdp.service

01 Linux配置



基础版本：Linux version 4.18.0-310.el8.x86_64 (CentOS-8.4.2105-x86_64-dvd1.iso   )

> ​	cat /proc/version

 关闭黑屏

> ​	setting>power>Blank screen

关闭锁屏

> ​	setting>privacy>screen lock

免登陆

> ​	setting>detail>user>unlock>Automatic Login

Vmware设置共享文件夹

[Vmware共享文件夹](https://docs.vmware.com/cn/VMware-Workstation-Player-for-Windows/16.0/com.vmware.player.win.using.doc/GUID-D6D9A5FD-7F5F-4C95-AFAB-EDE9335F5562.html#:~:text=%E8%BF%87%E7%A8%8B%201%20%E9%80%89%E6%8B%A9%E8%99%9A%E6%8B%9F%E6%9C%BA%EF%BC%8C%E7%84%B6%E5%90%8E%E9%80%89%E6%8B%A9Player%20%3E%20%E7%AE%A1%E7%90%86%20%3E%20%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%AE%BE%E7%BD%AE%E3%80%82%202,%E7%9B%AE%E5%BD%95%EF%BC%8C%E8%AF%B7%E9%80%89%E6%8B%A9%E5%9C%A8Windows%20%E5%AE%A2%E6%88%B7%E6%9C%BA%E4%B8%AD%E6%98%A0%E5%B0%84%E4%B8%BA%E7%BD%91%E7%BB%9C%E9%A9%B1%E5%8A%A8%E5%99%A8%E3%80%82%20...%205%20%E5%8D%95%E5%87%BB%E6%B7%BB%E5%8A%A0%E4%BB%A5%E6%B7%BB%E5%8A%A0%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E5%A4%B9%E3%80%82%20...%206%20%E6%B5%8F%E8%A7%88%E5%88%B0%E6%88%96%E9%94%AE%E5%85%A5%E4%B8%BB%E6%9C%BA%E7%B3%BB%E7%BB%9F%E4%B8%8A%E8%A6%81%E5%85%B1%E4%BA%AB%E7%9A%84%E7%9B%AE%E5%BD%95%E8%B7%AF%E5%BE%84)

terminal快捷方式

setting>device>keyboard>add

名字就填：terminal
命令填入：/usr/bin/gnome-terminal



#### centos8 解决 “Failed to set locale, defaulting to C.UTF-8”

> 	解决方案
> 方案一：设置系统环境变量
> echo "export LC_ALL=en_US.UTF-8"  >>  /etc/profile
> source /etc/profile
> 
> 
> 方案二：设置个人环境变量
> echo "export LC_ALL=en_US.UTF-8"  >>  ~/.bashrc
> source ~/.bashrc
> 

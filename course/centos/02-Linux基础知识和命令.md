

# 02-Linux基础知识和命令

### 1.基本格式及命令

## 软件仓库

1.切换软件仓库

/etc/yum.repos.d/Centos-Base.repo

https://centos.org/download/mirrors/ 所有镜像位置

https://blog.csdn.net/xiaoyu19910321/article/details/102752135 教程

yum update /upgrade 两个命令近乎没有什么区别

sudo yum rpm -i *.rpm 安装软件包

sudo yum  rpm -e 包名   用于卸载

sudo yum  yum localinstall *.rpm  手动安装本地rpm

yum remove/autoremove  卸载软件

## 手册

sudo yum install -y man-pages 安装手册

sudo mandb  更新最新手册

####  1.1 参数格式

​	短参数 command -p 10    - 参数空格值

   参数 command --paramter=10    --参数=值

#### 1.2 常用命令

​		Ctrl+L  ==clear 清屏

​		Ctrl + D 关闭终端

​		Shift + pgUp/PgDown  向上向下滚屏

​		Ctrl + A /E 光标调到开头/结尾

​		Ctrl+U/K 删除光标 左侧/右侧命令字符 

​		Ctrl+W 删除左边第一个单词

​		Ctrl+Y粘贴 Ctrl+U Ctrl+K或Ctrl+W删除的字符串

####  1.3 man

​	功能：查看指令说明

​    安装: yum install -y man man-pages

   基础命令：man cd

### 2. tty

​    功能：进入完全终端状态（terminal）

####     2.1 基本使用： 

​	进入： Ctrl + Alt + F2~F6(tty1~tty5)

​	退出： Ctrl + Alt + F2

### 3. ls 及 ll

​	功能： 展示整个目录结构

#### 3.1 Linux 目录结构

> ​	bin  “binary” 二进制执行文件
>
> dev “device”  设备 包含外设
>
> etc "and so on " 配置文件
>
> home  用户的私人文件 /home/用户
>
> lib "library" 被程序所调用的库文件
>
> media "媒体" 可移动的外设
>
> mnt "mount" 挂载 临时挂载
>
> opt "可选的应用软件包"
>
> root 超级用户根目录
>
> sbin "system binaray" 系统二进制执行文件
>
> srv "service" 包含一些网络服务启动之后所需要取用的数据
>
> tmp "temporary" 临时数据
>
> usr "Unix Software Resource" Unix操作系统软件资源
>
> 类型“C:\windows和C:\Program Files”两个文件夹的集合 安装了大部分调用的程序
>
> var "variable" 动态的可变的

#### 	3.1 基本使用

| 命令     | 功能                     | 备注         |
| -------- | ------------------------ | ------------ |
| ls       | 查看目录结构             |              |
| ls -a    | 隐藏文件所有             |              |
| ls -A    | 藏文件所有 除 .和..      |              |
| ls -l    | 详情                     | 等同于 ll    |
| ls -h    | 以ko.mo,go的形式显示大小 |              |
| ls -t    | 按照时间最近修改排序     |              |
| ls -laht |                          | 组合叠加使用 |

### 4. which

​	功能：获取命令可执行文件的位置

​	基本命令： which pwd   ==>/usr/bin/pwd

### 5. cd

​	功能： 切换目录

​    基本命令： cd /etc

### 6. du

   功能： 显示目录包含文件的大小

  基本命令

| 命令  | 功能                     | 备注 |
| ----- | ------------------------ | ---- |
| du    | 显示目录包含文件的大小   |      |
| du -h | 以ko.mo,go的形式显示大小 |      |
| du -a | 显示目录和文件大小       |      |
| du -s | 只显示总计大小           |      |
|       |                          |      |

### 7. cat

功能： 浏览文件 一次性浏览完成

| 命令   | 功能           | 备注 |
| ------ | -------------- | ---- |
| cat    | 一次性显示完成 |      |
| cat -n | 显示行号       |      |

### 8. less 

功能：分页显示文件内容

基本命令：less file.txt  空格键翻屏

阅读时命令

| 命令         | 功能                      | 备注                  |
| ------------ | ------------------------- | --------------------- |
| d            | 前进半个屏幕              |                       |
| b 或者PageUp | 后退一页                  |                       |
| y            | 上一行                    |                       |
| u            | 后退半个屏幕              |                       |
| q            | 退出                      |                       |
| =            | 显示文件位置 当前在文件的 |                       |
| /            | 进入搜索模式              | n下个位置 N上一个位置 |

### 9. head 

功能：默认显示前10行

基础命令: head file.txt

| 命令    | 功能     | 备注    |
| ------- | -------- | ------- |
| head    |          |         |
| head -n | 显示几行 | n是数字 |

### 10. tail

功能：显示尾

基础命令： tail file.txt

| 命令    | 功能                                            | 备注    |
| ------- | ----------------------------------------------- | ------- |
| tail    |                                                 |         |
| tail -n | 结尾几行                                        | n是数字 |
| tail -f | 实时追踪文件更新                                |         |
| tail -s | 追踪文件间隔秒数  tail -f -s 4 xxx  4秒检测一次 |         |

### 11. touch

功能：创建新文件

基础命令： touch new_file1  new_file2 空格隔开如果名称有空格需要用引号包裹

### 12. mkdir

功能：创建新目录

| 命令                   | 功能         | 备注 |
| ---------------------- | ------------ | ---- |
| mkdir  newDir          |              |      |
| mkdir -p one/two/three | 递归创建目录 |      |

### 13. cp

功能： 拷贝文件及目录

基础命令： copy file  toDir/

| 命令                  | 功能                    | 备注 |
| --------------------- | ----------------------- | ---- |
| copy file dir/        | 将file复制大屏dir目录下 |      |
| copy -r oldDIr newDir | 递归拷贝目录内容        |      |
|                       |                         |      |
|                       |                         |      |

### 14. mv 

功能：移动文件（类似剪切）可以当重命名

基本命令： mv file new_file

### 15. rm

功能：删除

基本命令： rm file.txt

| 命令            | 功能                    | 备注 |
| --------------- | ----------------------- | ---- |
| rm file_name    | 删除命令                |      |
| rm -i file_name | 每个文件删除都需要确认  |      |
| rm -f file_name | 强制删除                |      |
| rm -r dir_name  | 递归删除  删除目录      |      |
| rmdir  dir_name | 删除目录 只能删除空目录 |      |

### 16. ln

功能：链接 （快捷方式或功能指向）

基本命令：

| 命令              | 功能             | 备注                                                 |
| ----------------- | ---------------- | ---------------------------------------------------- |
| ln file1 file2    | 文件1硬链接file2 | 删除file1或者file2对其他文件没有影响                 |
| ln -s file1 file2 | 创建软链接       | 除file2 对file1 无影响， 删除file1 file2就找不到目标 |
|                   |                  |                                                      |
|                   |                  |                                                      |

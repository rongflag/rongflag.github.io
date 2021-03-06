## 03.Linux权限操作

### 1. sudo 暂时成为root权限

   功能：暂时成为root权限

   基础命令：sudo 命令

### 2. su 切换成root

 功能：切换成root用户

基础命令：su 输入密码 切换成root用户

exit退出

### 3. useradd 添加用户

功能：添加用户

基础命令：useradd 用户1  用户2

普通用户不能添加用户，需要用root权限才可以

### 4. usermod 修改用户

功能：修改用户

基础命令：usermod   -l 对用户重命名 /home下名称不改

usermod -g 群组名 用户名

-G 添加到多个群组  usermod -G 群组1,群组2 用户  逗号分隔没有空格

-aG追加群组  当前群组不变

### 5.password 修改密码

功能：修改用户密码

基础命令：password 用户  密码

### 6.userdel 删除用户

功能：删除用户

基础命令：userdel 用户名   删除用户 不删除/home下面目录

​                    userdel -r 用户名   删除用户  删除/home

### 7. groupadd 创建群组

​		功能：创建群组

​		基础命令：groupadd 群组名

> ​	用户添加时，不选群组时 默认创建与用户名相同的群组

### 8.group 查看

功能：查看用户所在群组

基础命令：group 用户名  查看用户所在群组

### 9.groupdel 删除群组

功能：删除群组

基础命令：groupdel  群组名词

### 10.chown 修改文件所有者

功能：修改文件所有者

基础命令： chown 用户 文件

chown 用户名:群组名 修改文件群组及所有者

chown -R R 用户名:群组 文件  递归修改目录及子文件

### 11. chgrp 修改文件群组

功能：修改文件所有群组

基础命令： chgrp 群组文件

### 12. chmod 修改文件访问权限

文件首字母 文件属性字段

| 字符   | 功能                |
| ------ | ------------------- |
| d      | 目录                |
| l      | 链接                |
| -      | 文件                |
| x      | 可执行文件          |
| d      | 目录                |
| p或者s | 与shell编程有关的文 |
| c      | 一个字符设备文件    |

文件树形所有字段意义

| d                                 | rwx    | rwx  | rwx      |
| --------------------------------- | ------ | ---- | -------- |
| 属性 d 目录 - 普通文件 l 链接文件 | 所有者 | 群组 | 其他用户 |

| 权限 | 数字 |
| ---- | ---- |
| r    | 4    |
| w    | 2    |
| x    | 1    |

合并权限就相加就行，例如数字6具有读和写的权限

| 权限 | 数字 | 计算  |
| ---- | ---- | ----- |
| ---  | 0    | 0 0 0 |
| r--  | 4    | 4 0 0 |
| -w-  | 2    | 0 2 0 |
| --r  | 1    | 0 0 1 |
| rw-  | 6    | 4 2 0 |
| -wx  | 3    | 0 2 1 |
| r-x  | 5    | 4 0 1 |
| rwx  | 7    | 4 2   |

#### 1. 权限用数字分配

chmod 777 最大的权限

#### 2.权限用字母分配：

用字母用法

| 字母 | 意思  |          |
| ---- | ----- | -------- |
| u    | user  | 用户     |
| g    | gtoup | 群组     |
| o    | other | 其他用户 |
| a    | all   | 所有用户 |
| +    |       | 添加权限 |
| -    |       | 去除权限 |
| =    |       | 分配权限 |

chmod u+rx file  文件file的所有者增加读和运行的权限

chmod g+r   file    文件file的群组其他增加读权限

chmod o-r file  文件file其他用户移除读的权限

chmod g+r o-r file 文件file的群组其他用户增加读权限，其他用户移除读权限

chmod +x file 文件file未所有用户增加执行权限

chmod u=rwx,g=r,o=- file   文件file所有者有读写执行权限，群组其他用户读权限，其他用户没有权限

-R 递归文件夹子文件夹文件权限操作

chmod -R 700 /文件夹/文件夹


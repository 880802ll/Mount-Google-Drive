# VPS挂载Google Drive教程

**系统要求：CentOS+、Debian7+、Ubuntu12+**

## 一、安装&配置rclone

1. **安装rclone：**

``` 
curl https://rclone.org/install.sh | sudo bash
```

2. **配置**

```
rclone config
```

------

```
o remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n

name emby 随意取 但是后面的代码也会相应改变
```

------

```
Choose a number from below, or type in your own value
选谷歌 （13） 可能会因为版本的不同而改变 注意选择 Google drive
```

------

```
client_id> 直接回车
```

------

```
client_secret> 直接回车
```

------

```
Choose a number from below, or type in your own value
scope> 1 选1 有最大的使用权限。
```

------

```
ID of the root folder
Leave blank normally.

Fill in to access "Computers" folders (see docs), or for rclone to use
a non root folder as its starting point.

Note that if this is blank, the first time rclone runs it will fill it
in with the ID of the root folder.

Enter a string value. Press Enter for the default ("").
root_folder_id> 空，直接回车。空是跟路径如果想用别的根路径
```

------

```
Service Account Credentials JSON file path
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file> 直接回车
```

------

```
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> n 不用别的高级配置
```

------

```
Use auto config?
Say Y if not sure
Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n> n 因为我们是vps操作，不能auto config
```

------

```
Please go to the following link: https://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Log in and authorize rclone for access
Enter verification code>贴入你获取到的key
```

------

```
Configure this as a team drive?
y) Yes
n) No (default)
y/n> y
```

------

```
Choose a number from below, or type in your own value
1 / xxxxx
\ "0AGfwXXXXXXXXXXXX"
2 / yyyyy
\ "0AXXXXXXXXXXXXXXX"
Enter a Team Drive ID>
```

------

```
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
```

------

```
e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```



## 二、挂载磁盘

1. **挂载**

- **创建过载目录**

```
mkdir -p /home/gdrive
```

------

- **手动挂载磁盘（全部复制直接粘贴）**

```
/usr/bin/rclone mount emby: /home/gdrive \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --buffer-size 32M \
 --dir-cache-time 12h \
 --vfs-read-chunk-size 64M \
 --vfs-read-chunk-size-limit 1G &   
```

**手动挂载成功后会出现[1] xxxxx进程成功提示，可以查看挂载目录是否正常**

```
df -h
```

**出现1.0P的盘符即为挂载成功**

**手动挂载若报错executable file not found in $PATH**

**可以安装fuse**

```
yum install fuse -y
```

2. **配置自动挂载**

- **配置自动挂载文件（全部复制直接粘贴）**

```
cat > /etc/systemd/system/rclone.service <<EOF
[Unit]
Description=Rclone
AssertPathIsDirectory=LocalFolder
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount emby: /volume2/Movies/gdrive \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --buffer-size 32M \
 --dir-cache-time 12h \
 --vfs-read-chunk-size 64M \
 --vfs-read-chunk-size-limit 1G
ExecStop=/bin/fusermount -u LocalFolder
Restart=on-abort
User=root

[Install]
WantedBy=default.target
EOF
```

- **启动配置文件**

```
systemctl start rclone
```

- **开机自启**

```
systemctl enable rclone
```

3. **配置虚拟内存**

``` 
wget https://www.moerats.com/usr/shell/swap.sh && bash swap.sh
```

## 三、其他

**[原文教程地址](https://github.com/bigdongdongCLUB/welcome/issues/6)**


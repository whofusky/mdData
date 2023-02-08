# RHEL 6.7 ftp账号配置

此文件介绍的在Red Hat Enterpsise Linux Server release 6.7 操作系统上配置的结果；其他操作系统可以借鉴但不确定是否与当前文档描述的一致；

文档内容分为三部分:
```

    一. /etc/vsftpd/vsftpd.conf文件修改项

    二. 允许本操作系统用户登录ftp涉及的文件

    三. 设置ftp用户可以访问的根目录

```
   
说明:    
```

 1. 此文件举例用的账号是`ftp_yc1`
 2. 此文件的所有操作需要用root用户进行
 3. 所有配置文件修改后需要重启vsftpd服务: service vsftpd restart

```
     
</br>

------------------------------------------------------------------------------

</br>

## 一、 vsftpd.conf添加或修改项

> 打开/etc/vsftpd/vsftpd.conf配置文件   
>> 此配置文件配置格式说明:   
>>> 1. 行首有#号的说明当前是`注释`行(即未生效或注释说明行)
>>> 2. 配置的key和value之间等号左右尽量不要有空格

#### 1. 检查配置文件中是否有`userlist_deny`的配置项

  (1). 如果没有此配置项，直接忽略此项的设置即可；    

  (2). 如果有此设置项，则需要将值设置成如下形式:
```
userlist_deny=YES
```

#### 2. 添加或删除行首注释的值

在文件或查找如下键值，如果是注释掉的去掉行首的注释#号，如果值不同的设置如下的值，如果没有的则新增加如下键值

```
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
user_config_dir=/etc/vsftpd/user_conf
```

> 注意:     
>   1. 如果没有/etc/vsftpd/chroot_list文件则需要用手动建立一个空文件；
>   1. 如果没有/etc/vsftpd/user_conf文件夹则需要用手动建立一个空目录

#### 3. 注释掉tcp_wrappers配置

如果没有此配置忽略即可。


</br>

------------------------------------------------------------------------------

</br>


## 二、 允许操作系统本地用户作为ftp账号登录


#### 1. 修改ftpusers文件

> 文件位置: /etc/vsftpd/ftpusers   

打开此文件 查找是否有`ftp_yc1` 用户，如果有注释掉或删除即可

#### 2. 修改user_list文件

> 文件位置: /etc/vsftpd/user_list   

打开此文件 查找是否有`ftp_yc1` 用户，如果有注释掉或删除即可

</br>

------------------------------------------------------------------------------

</br>


## 三、 设置ftp用户的根目录(选做)

```

注意:
    1. 此设置是选做，如果不做此部分的设置，用户也能登录ftp服务器，但
       用户可以访问服务器上的任何目录（只要此用户有相应的操作系统权限）

    2. 如果需要设置或简化用户登录下载文件目录的长度则需要进行此设置

    3. 此设置的效果为: 假定设置ftp_yc1用户根目录为/zfmd/tmp,其中
       tmp的目录结构为:    
        tmp/
        ├── dir1/
        ├── dir2/
        └── dir3/
        那么用ftp_yc1登录服务器后只能看到dir1,dir2,dir3;
        如果dir1有文件1.txt则用命令get /dir1/1.txt即可取到文件;
        完全屏蔽了dir1上级目录在操作系统中的存在。

```


#### 1. 修改chroot_list文件

> 文件位置: /etc/vsftpd/chroot_list   

打开此文件添加一行内容:ftp_yc1 
> 文件内容就是账号名  
> 如果已有忽略即可


#### 2. 修改或添加user_conf/ftp_yc1文件

````
    注意:
        如果此部分不设置,默认根目录为用户的HOME目录
````

> 文件位置: /etc/vsftpd/user_conf/ftp_yc1

其中ftp_yc1是账号也是文件名,如果没有此文件手动建立即可

打开此文件添加local_root值的设置，其值为根目录的绝对值路径

```
    例如: 要设置/zfmd/data为根目录则文件内容为
local_root=/zfmd/data
```
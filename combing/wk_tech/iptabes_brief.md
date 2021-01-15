iptabes_brief





# iptables 简要整理

-----

## 目录

* [1. 命令格式](#命令格式)
	* [1.1 一般形式](#一般形式)
	* [1.2 简单的查看iptables表的规则](#简单的查看iptables表的规则)
	
* [2. 防火墙相关概念](#防火墙相关概念)
	* [2.1 链](#链)
	* [2.2 表](#表)
	* [2.3 表链关系](#表链关系)

* [3. iptables规则管理](#iptables规则管理)
	* [3.1 添加规则](#添加规则)
	* [3.2 删除规则](#删除规则)
	* [3.3 修改规则](#修改规则)
	* [3.4 保存规则](#保存规则)
	
* [4. iptables匹配条件总结之一](#iptables匹配条件总结之一)
	* [4.1 基本匹配条件](#基本匹配条件)
	* [4.2 扩展匹配条件](#扩展匹配条件)
		* [4.2.1 tcp扩展模块](#tcp扩展模块)
		* [4.2.2 multiport扩展模块](#multiport扩展模块)
		* [4.2.3 iprange模块](#iprange模块)
		* [4.2.4 string模块](#string模块)
		* [4.2.5 time模块](#time模块)
		* [4.2.6 connlimit 模块](#connlimit模块)
		* [4.2.7 limit模块](#limit模块)
		* [4.2.8 –tcp-flags](#4d2d8)
		* [4.2.9 之udp扩展与icmp扩展](#之udp扩展与icmp扩展)
		* [4.2.10 扩展模块之state扩展](#扩展模块之state扩展)

* [5. 黑白名单机制](#黑白名单机制)

* [6. 自定义链](#自定义链)
	* [6.1 创建自定义链](#创建自定义链)
	* [6.2 自定义链中配置规则](#自定义链中配置规则)
	* [6.3 引用自定义链](#引用自定义链)
	* [6.4 重命名自定义链](#重命名自定义链)
	* [6.5 删除自定义链](#删除自定义链)
	
* [7. 网络防火墙](#网络防火墙)
	* [7.1 开启转发](#开启转发)
	* [7.2 配置转发](#配置转发)
	
* [8. 动作总结之一](#动作总结之一)
	* [8.1 动作REJECT](#动作REJECT)
	* [8.2 动作LOG](#动作LOG)

* [9. 动作总结之二](#动作总结之二)
	* [9.1 SNAT相关操作](#SNAT相关操作)
	* [9.2 DNAT相关操作](#DNAT相关操作)
	* [9.3 REDIRECT动作](#REDIRECT动作)

* [10. 常用套路](#常用套路)



---

## 1
## 命令格式 

### 1.1
### 一般形式

```shell
iptables [-t 表] -命令 匹配   操作
  命令                说明

  -P  --policy        <链名>  定义默认策略
  -L  --list          <链名>  查看iptables规则列表
  -A  --append        <链名>  在规则列表的最后增加1条规则
  -I  --insert        <链名>  在指定的位置插入1条规则
  -D  --delete        <链名>  从规则列表中删除1条规则
  -R  --replace       <链名>  替换规则列表中的某条规则
  -F  --flush         <链名>  删除表中所有规则
  -Z  --zero          <链名>  将表中数据包计数器和流量计数器归零
  -X  --delete-chain  <链名>  删除自定义链
  -v  --verbose       <链名>  与-L他命令一起使用显示更多更详细的信息
```
    
1. 匹配条件分为基本匹配条件与扩展匹配条件 

2. 处理动作:(常用的动作) 

-   ACCEPT：允许数据包通过。
  
-   DROP：直接丢弃数据包，不给任何回应信息，这时候客户端会感觉自己的请求泥牛入海了，过了超时时间才会有反应。

-   REJECT：拒绝数据包通过，必要时会给数据发送端一个响应的信息，客户端刚请求就会收到拒绝的信息。

-   SNAT：源地址转换，解决内网用户用同一个公网地址上网的问题。

-   MASQUERADE：是SNAT的一种特殊形式，适用于动态的、临时会变的ip上。

-   DNAT：目标地址转换。

-   REDIRECT：在本机做端口映射。

-   LOG：在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则，也就是说除了记录以外不对数据包做任何其他操作，仍然让下一条规则去匹配。

### 1.2
### 简单的查看iptables表的规则

```shell
iptables -t 表名 -L
   查看对应表的所有规则，-t选项指定要操作的表，省略"-t 表名"时，默认表示操作filter表，-L表示列出规则，即查看规则。
iptables -t 表名 -L 链名
iptables -t 表名 -v -L
iptables -t 表名 -n -L
iptables --line-numbers -t 表名 -
iptables -t 表名 -v -x -L
iptables --line -t filter -nvxL
iptables --line -t filter -nvxL INPUT
```   


> [返回目录](#目录)

        
----

## 2
## 防火墙相关概念

**从逻辑上讲**防火墙可以大体分为主机防火墙和网络防火墙。

**从物理上讲**防火墙可以分为硬件防火墙和软件防火墙。

iptables其实**不是真正的防火墙**，我们可以把它理解成一个客户端代理，用户通过iptables这个代理，将用户的安全设定执行到对应的"安全框架"中，这个"安全框架"才是真正的防火墙，这个框架的名字叫netfilter，所以说，虽然我们使用service iptables start启动iptables"服务"，但是其实准确的来说，iptables并没有一个守护进程，所以并不能算是真正意义上的服务，而应该算是内核提供的功能。

### 2.1
### 链

 到本机某进程的报文：PREROUTING --> INPUT
 
 由本机转发的报文：PREROUTING --> FORWARD --> POSTROUTING
 
 由本机的某进程发出报文（通常为响应报文）：OUTPUT --> POSTROUTING   
   
### 2.2
### 表

-   filter表：负责过滤功能，防火墙；内核模块：iptables_filter   
-   nat表：network address translation，网络地址转换功能；内核模块：iptable_nat   

-   mangle表：拆解报文，做出修改，并重新封装 的功能；iptable_mangle   

- raw表：关闭nat表上启用的连接追踪机制；iptable_raw

### 2.3
### 表链关系

 -   PREROUTING 的规则可以存在于：raw表，mangle表，nat表。
 
 -   INPUT 的规则可以存在于：mangle表，filter表，（ **centos7中还有nat表**，c entos6中没有）。
 
 -   FORWARD 的规则可以存在于：mangle表，filter表。
 
 -   OUTPUT 的规则可以存在于：raw表mangle表，nat表，filter表。
 
 -   POSTROUTING 的规则可以存在于：mangle表，nat表。     

 -   raw 表中的规则可以被哪些链使用：PREROUTING，OUTPUT
 
 -   mangle 表中的规则可以被哪些链使用：PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING
 
 -   nat 表中的规则可以被哪些链使用：PREROUTING，OUTPUT，POSTROUTING（centos7中还有INPUT，centos6中没有）
 
 -   filter 表中的规则可以被哪些链使用：INPUT，FORWARD，OUTPUT
   
 **优先级次序**（由高而低）：   
 raw --> mangle --> nat --> filter   
 



> [返回目录](#目录)


-----

## 3
## iptables规则管理

### 3.1
### 添加规则   

1. `命令语法：iptables -t 表名 -A 链名 匹配条件 -j 动作`   
    -A选项表示在对应链的末尾添加规则，省略-t选项时，表示默认操作filter表中的规则   
    示例：iptables -t filter -A INPUT -s 192.168.1.146 -j DROP
     
2. `命令语法：iptables -t 表名 -I 链名 匹配条件 -j 动作`    	 
    -I选型表示在对应链的开头添加规则    
    示例：iptables -t filter -I INPUT -s 192.168.1.146 -j ACCEPT

3. `命令语法：iptables -t 表名 -I 链名 规则序号 匹配条件 -j 动作`    
    在指定表的指定链的指定位置添加一条规则     
    示例：iptables -t filter -I INPUT 5 -s 192.168.1.146 -j REJEC

4. `命令语法：iptables -t 表名 -P 链名 动作`    
    设置指定表的指定链的默认策略（默认动作），并非添加规则    
    示例：iptables -t filter -P FORWARD ACCEPT   

### 3.2
### 删除规则

注意点：如果没有保存规则，删除规则时请慎重

1. `命令语法：iptables -t 表名 -D 链名 规则序号`   
	按照规则序号删除规则，删除指定表的指定链的指定规则，-D选项表示删除对应链中的规则。   
	示例：iptables -t filter -D INPUT 3

2. `命令语法：iptables -t 表名 -D 链名 匹配条件 -j 动作`   
    按照具体的匹配条件与动作删除规则，删除指定表的指定链的指定规则。   
    示例：iptables -t filter -D INPUT -s 192.168.1.146 -j DROP
 
3. `命令语法：iptables -t 表名 -F 链名`   
    删除指定表的指定链中的所有规则，-F选项表示清空对应链中的规则，执行时需三思。    
    示例：iptables -t filter -F INPUT

4. `命令语法：iptables -t 表名 -F`    
    删除指定表中的所有规则，执行时需三思。     
    示例：iptables -t filter -F
    
### 3.3
### 修改规则
    
1. `命令语法：iptables -t 表名 -R 链名 规则序号 规则原本的匹配条件 -j 动作`	   
    修改指定表中指定链的指定规则，-R选项表示修改对应链中的规则，使用-R选项时要同时指定对应的链以及规则对应的序号，并且规则中原本的匹配条件不可省略。     
    示例：iptables -t filter -R INPUT 3 -s 192.168.1.146 -j ACCEPT
    
2. 其他修改规则的方法：先通过编号删除规则，再在原编号位置添加一条规则。
  
3. `命令语法：iptables -t 表名 -P 链名 动作`   
    修改指定表的指定链的默认策略（默认动作），并非修改规则，可以使用如下命令    
    示例：iptables -t filter -P FORWARD ACCEPT

### 3.4
### 保存规则
    
表示将iptables规则保存至/etc/sysconfig/iptables文件中，如果对应的操作没有保存，那么当重启iptables服务以后
        
1.  service iptables save
    
2.  centos7中使用默认使用firewalld，如果想要使用上述命令保存规则，需要安装iptables-services,或者使用如下方法保存规则     
  `iptables-save > /etc/sysconfig/iptable`    
    可以使用如下命令从指定的文件载入规则，注意：重载规则时，文件中的规则将会覆盖现有规则。   
    `iptables-restore < /etc/sysconfig/iptables`
    


> [返回目录](#目录)


-----

## 4
## iptables匹配条件总结之一

### 4.1
### 基本匹配条件
    
1.   -s 用于匹配报文的源地址,可以同时指定多个源地址，每个IP之间用逗号隔开，也可以指定为一个网段。     
    #示例如下    
    iptables -t filter -I INPUT -s 192.168.1.111,192.168.1.118 -j DROP    
    iptables -t filter -I INPUT -s 192.168.1.0/24 -j ACCEPT
    iptables -t filter -I INPUT ! -s 192.168.1.0/24 -j ACCEPT
    
2.   -d 用于匹配报文的目标地址,可以同时指定多个目标地址，每个IP之间用逗号隔开，也可以指定为一个网段。   
    #示例如下   
    iptables -t filter -I OUTPUT -d 192.168.1.111,192.168.1.118 -j DROP   
    iptables -t filter -I INPUT -d 192.168.1.0/24 -j ACCEPT   
    iptables -t filter -I INPUT ! -d 192.168.1.0/24 -j ACCEPT   
    
3.   -p 用于匹配报文的协议类型,可以匹配的协议类型tcp、udp、udplite、icmp、esp、ah、sctp等（centos7中还支持icmpv6、mh）。   
    #示例如下   
    iptables -t filter -I INPUT -p tcp -s 192.168.1.146 -j ACCEPT   
    iptables -t filter -I INPUT ! -p udp -s 192.168.1.146 -j ACCEPT   
    
4.   -i 用于匹配报文是从哪个网卡接口流入本机的，由于匹配条件只是用于匹配报文流入的网卡，所以在OUTPUT链与POSTROUTING链中不能使用此选项。   
    #示例如下   
    iptables -t filter -I INPUT -p icmp -i eth4 -j DROP   
    iptables -t filter -I INPUT -p icmp ! -i eth4 -j DROP   
    
5.   -o 用于匹配报文将要从哪个网卡接口流出本机，于匹配条件只是用于匹配报文流出的网卡，所以在INPUT链与PREROUTING链中不能使用此选项。   
    #示例如下   
    iptables -t filter -I OUTPUT -p icmp -o eth4 -j DROP   
    iptables -t filter -I OUTPUT -p icmp ! -o eth4 -j DROP   


> [返回目录](#目录)


### 4.2
### 扩展匹配条件

#### 4.2.1
#### tcp扩展模块

常用的扩展匹配条件如下：   
`-p tcp -m tcp --sport` 用于匹配tcp协议报文的源端口，可以使用冒号指定一个连续的端口范围   
`-p tcp -m tcp --dport` 用于匹配tcp协议报文的目标端口，可以使用冒号指定一个连续的端口范围   
    
    #示例如下
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p tcp -m tcp --sport 22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport 22:25 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport :22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport 80: -j REJECT
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p tcp -m tcp ! --sport 22 -j ACCEP
 
> [返回目录](#目录)

   
#### 4.2.2 
#### multiport扩展模块
    常用的扩展匹配条件如下：
    -p tcp -m multiport --sports 用于匹配报文的源端口，可以指定离散的多个端口号,端口之间用"逗号"隔开
    -p udp -m multiport --dports 用于匹配报文的目标端口，可以指定离散的多个端口号，端口之间用"逗号"隔开
    
    #示例如下
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p udp -m multiport --sports 137,138 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 22,80 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport ! --dports 22,80 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 80:88 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 22,80:88 -j REJECT

> [返回目录](#目录)

       
#### 4.2.3 
#### iprange模块
    
包含的扩展匹配条件如下

`--src-range`：指定连续的源地址范围   
`--dst-range`：指定连续的目标地址范围   
	
    #示例
    iptables -t filter -I INPUT -m iprange --src-range 192.168.1.127-192.168.1.146 -j DROP
    iptables -t filter -I OUTPUT -m iprange --dst-range 192.168.1.127-192.168.1.146 -j DROP
    iptables -t filter -I INPUT -m iprange ! --src-range 192.168.1.127-192.168.1.146 -j DRO

#### 4.2.4
#### string模块
    
常用扩展匹配条件如下

`--algo`：指定对应的匹配算法，可用算法为bm、kmp，此选项为必需选项。   
`--string`：指定需要匹配的字符串   
    #示例   
    iptables -t filter -I INPUT -p tcp --sport 80 -m string --algo bm --string "OOXX" -j REJECT   
    iptables -t filter -I INPUT -p tcp --sport 80 -m string --algo bm --string "OOXX" -j REJEC   

> [返回目录](#目录)

   
#### 4.2.5    
#### time模块
    
    常用扩展匹配条件如下
    --timestart：用于指定时间范围的开始时间，不可取反
    --timestop：用于指定时间范围的结束时间，不可取反
    --weekdays：用于指定"星期几"，可取反
    --monthdays：用于指定"几号"，可取反
    --datestart：用于指定日期范围的开始日期，不可取反
    --datestop：用于指定日期范围的结束时间，不可取反
    #示例
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --timestart 09:00:00 --timestop 19:00:00 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 443 -m time --timestart 09:00:00 --timestop 19:00:00 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --weekdays 6,7 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --monthdays 22,23 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time ! --monthdays 22,23 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --timestart 09:00:00 --timestop 18:00:00 --weekdays 6,7 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --weekdays 5 --monthdays 22,23,24,25,26,27,28 -j REJECT
    iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --datestart 2017-12-24 --datestop 2017-12-27 -j REJECT

> [返回目录](#目录)

   
#### 4.2.6    
#### connlimit模块
    
    常用的扩展匹配条件如下
    --connlimit-above：单独使用此选项时，表示限制每个IP的链接数量。
    --connlimit-mask：此选项不能单独使用，在使用--connlimit-above选项时，配合此选项，则可以针对"某类IP段内的一定数量的IP"进行连接数量的限制，如果不明白可以参考
    上文的详细解释。
    #示例
    iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
    iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 20 --connlimit-mask 24 -j REJECT
    iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 10 --connlimit-mask 27 -j REJECT

#### 4.2.7    
#### limit模块
    
    常用的扩展匹配条件如下
    --limit-burst：类比"令牌桶"算法，此选项用于指定令牌桶中令牌的最大数量，上文中已经详细的描述了"令牌桶"的概念，方便回顾。
    --limit：类比"令牌桶"算法，此选项用于指定令牌桶中生成新令牌的频率，可用时间单位有second、minute 、hour、day
    
    #示例 #注意，如下两条规则需配合使用，具体原因上文已经解释过，忘记了可以回顾。
    iptables -t filter -I INPUT -p icmp -m limit --limit-burst 3 --limit 10/minute -j ACCEPT
    iptables -t filter -A INPUT -p icmp -j REJECT
    

> [返回目录](#目录)

   
#### 4.2.8
### 4d2d8
#### –tcp-flags
      
    用于匹配报文的tcp头的标志位
    
    #示例
    iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN -j REJECT
    iptables -t filter -I OUTPUT -p tcp -m tcp --sport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN,ACK -j REJECT
    iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags ALL SYN -j REJECT
    iptables -t filter -I OUTPUT -p tcp -m tcp --sport 22 --tcp-flags ALL SYN,ACK -j REJECT

   
**--syn**
    
    用于匹配tcp新建连接的请求报文，相当于使用"--tcp-flags SYN,RST,ACK,FIN SYN"
    
    #示例
    iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --syn -j REJECT

> [返回目录](#目录)

   
#### 4.2.9     
#### 之udp扩展与icmp扩展
   
**udp扩展**
    
    常用的扩展匹配条件
    --sport：匹配udp报文的源地址
    --dport：匹配udp报文的目标地址
    
    #示例
    iptables -t filter -I INPUT -p udp -m udp --dport 137 -j ACCEPT
    iptables -t filter -I INPUT -p udp -m udp --dport 137:157 -j ACCEPT
    #可以结合multiport模块指定多个离散的端口


    
**icmp扩展**
    
    常用的扩展匹配条件
    --icmp-type：匹配icmp报文的具体类型
    
    #示例
    iptables -t filter -I INPUT -p icmp -m icmp --icmp-type 8/0 -j REJECT
    iptables -t filter -I INPUT -p icmp --icmp-type 8 -j REJECT
    iptables -t filter -I OUTPUT -p icmp -m icmp --icmp-type 0/0 -j REJECT
    iptables -t filter -I OUTPUT -p icmp --icmp-type 0 -j REJECT
    iptables -t filter -I INPUT -p icmp --icmp-type "echo-request" -j REJECT

    
**icmp报文被细分为如下各种类型**
    
    icmp match options:
    [!] --icmp-type typename        match icmp type
    [!] --icmp-type type[/code]     (or numeric type or type/code)
    

|type|code|description|query|error| 
|----|----|----------|------|------|
|0| |echo-reply (pong) |* | |
| 3| |destination-unreachable | | * |
| | 0| network-unreachable | | * |
| | 1 | host-unreachable| | * |
| | 2 | protocol-unreachable | | * |
| | 3 | port-unreachable | |  *   |
| | 4 | fragmentation-needed | | * |
| | 5 | source-route-failed | | * |
| | 6 | network-unknown | | * |
| | 7 | host-unknown | |  *   |
| | 9 | network-prohibited | |  *   |
| | 10 | host-prohibited | |  *   |
| | 11 | TOS-network-unreachable | | * |
| | 12 | TOS-host-unreachable | |  * |
| | 13 | communication-prohibited | |  *   |
| | 14 | host-precedence-violation  | |  *   |
| | 15 | precedence-cutoff| | * |
| 4 | 0 |  source-quench| |  *   |
| 5 | |  redirect| | |
| | 0 |network-redirect| |  *   |
| | 1 |host-redirect| |  *   |
| | 2 |TOS-network-redirect| |  *   |
| | 3 |TOS-host-redirect| | * |
| 8 | 0 |echo-request (ping)|  *   | |
| 9 | 0 | router-advertisement|  *   | |
| 10 | 0 |router-solicitation|  *   | |
| 11 | |time-exceeded (ttl-exceeded)|  |    |
| |   0|ttl-zero-during-transit|      |  *   |
| |   1|ttl-zero-during-reassembly |      |  *   |
| 12   | |parameter-problem| | |
| | 0 |ip-header-bad| | * |
| | 1 |required-option-missing| |  *   |
| 13 | 0 |timestamp-request| * | |
| 14 | 0 | timestamp-reply| * | |
| 17 | 0 |address-mask-request| * | |
| 18 | 0 |address-mask-reply| * | |
        

> [返回目录](#目录)

       
#### 4.2.10 
#### 扩展模块之state扩展
    
一说到连接，你可能会下意识的想到tcp连接，但是，对于state模块而言的"连接"并不能与tcp的"连接"画等号，在TCP/IP协议簇中，UDP和ICMP是
没有所谓的连接的，但是对于state模块来说，tcp报文、udp报文、icmp报文都是有连接状态的，我们可以这样认为，对于state模块而言，只要两台机器在"你来我往"的通信，就算建立起了连接.   

对于state模块的连接而言，"连接"其中的报文可以分为5种状态，报文状态可以为`NEW、ESTABLISHED、RELATED、INVALID、UNTRACKED`
    
iptables -t filter -I INPUT -m state --state RELATED ESTABLISHED -j ACCEPT


> [返回目录](#目录)


----

## 5
## 黑白名单机制
    
如果使用白名单机制，我们就要把所有人都当做坏人，只放行好人。

如果使用黑名单机制，我们就要把所有人都当成好人，只拒绝坏人。

白名单机制似乎更加安全一些，黑名单机制似乎更加灵活一些。
    
这就是默认策略设置为DROP的缺点，在对应的链中没有设置任何规则时，这样使用默认策略为DROP是非常不明智的，因为管理员也会把自己拒之门外，即使对应的链中存在放行规则，当我们不小心使用"iptables -F"清空规则时，放行规则被删除，则所有数据包都无法进入，这个时候就相当于给管理员挖了个坑，所以，我们如果想要使用"白名单"的机制，最好将链的默认策略保持为"ACCEPT"，然后将"拒绝所有请求"这条规则放在链的尾部，将"放行规则"放在前面，这样做，既能实现"白名单"机制，又能保证在规则被清空
时，管理员还有机会连接到主机，示例如下。
    
    iptables -P INPUT   ACCEPT
    iptables -I INPUT -p tcp -m tcp --dport 22 -j ACCEPT
    iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    iptables -A INPUT -j REJECT


> [返回目录](#目录)

    
----

## 6
## 自定义链
    
原因如下：

    当默认链中的规则非常多时，不方便我们管理。
    想象一下，如果INPUT链中存放了200条规则，这200条规则有针对httpd服务的，有针对sshd服务的，有针对私网IP的，有针对公网IP的，假如，我们突然想要修改针对httpd服
    务的相关规则，难道我们还要从头看一遍这200条规则，找出哪些规则是针对httpd的吗？这显然不合理。
    所以，iptables中，可以自定义链，通过自定义链即可解决上述问题。

### 6.1
### 创建自定义链
    
    #示例：在filter表中创建IN_WEB自定义链
    iptables -t filter -N IN_WEB

### 6.2 
### 自定义链中配置规则
    
    iptables -t filter -I IN_WEB -s 192.168.1.139 -j REJECT
    iptables -I IN_WEB -s 192.168.1.199 -j REJECT

### 6.3
### 引用自定义链
    
    #示例：在INPUT链中引用刚才创建的自定义链
    iptables -t filter -I INPUT -p tcp --dport 80 -j IN_WEB

### 6.4    
### 重命名自定义链
    
    使用"-E"选项可以修改自定义链名，引用自定义链处的名称会自动发生改变。
    
    #示例：将IN_WEB自定义链重命名为WEB
    iptables -E IN_WEB WEB

### 6.5   
### 删除自定义链
    
    删除自定义链需要满足两个条件
    1、自定义链没有被引用
    2、自定义链中没有任何规则
    
    #示例：删除引用计数为0并且不包含任何规则的WEB链
    iptables -D INPUT 1
    iptables -t filter -F WEB
    iptables -t filter -X WEB
    #iptables -X WEB



> [返回目录](#目录)


-----

## 7
## 网络防火墙
    
    主机防火墙：针对于单个主机进行防护。
    网络防火墙： 往往处于网络入口或边缘，针对于网络入口进行防护，服务于防火墙背后的本地局域网。


### 7.1
### 开启转发
    
    #如果想要iptables作为网络防火墙，iptables所在主机开启核心转发功能，以便能够转发报文。
    #使用如下命令查看当前主机是否已经开启了核心转发，0表示为开启，1表示已开启
    cat /proc/sys/net/ipv4/ip_forward
    #使用如下两种方法均可临时开启核心转发，立即生效，但是重启网络配置后会失效。
    方法一：echo 1 > /proc/sys/net/ipv4/ip_forward
    方法二：sysctl -w net.ipv4.ip_forward=1
    #使用如下方法开启核心转发功能，重启网络服务后永久生效。
    配置/etc/sysctl.conf文件（centos7中配置/usr/lib/sysctl.d/00-system.conf文件），在配置文件中将 net.ipv4.ip_forward设置为1

### 7.2    
### 配置转发
    
    #由于iptables此时的角色为"网络防火墙"，所以需要在filter表中的FORWARD链中设置规则。
    #可以使用"白名单机制"，先添加一条默认拒绝的规则，然后再为需要放行的报文设置规则。
    #配置规则时需要考虑"方向问题"，针对请求报文与回应报文，考虑报文的源地址与目标地址，源端口与目标端口等。
    #示例为允许网络内主机访问网络外主机的web服务与sshd服务。
    
    iptables -A FORWARD -j REJECT
    iptables -I FORWARD -s 10.1.0.0/16 -p tcp --dport 80 -j ACCEPT
    iptables -I FORWARD -d 10.1.0.0/16 -p tcp --sport 80 -j ACCEPT
    iptables -I FORWARD -s 10.1.0.0/16 -p tcp --dport 22 -j ACCEPT
    iptables -I FORWARD -d 10.1.0.0/16 -p tcp --sport 22 -j ACCEPT
    
    #可以使用state扩展模块，对上述规则进行优化，使用如下配置可以省略许多"回应报文放行规则"。
    iptables -A FORWARD -j REJECT
    iptables -I FORWARD -s 10.1.0.0/16 -p tcp --dport 80 -j ACCEPT
    iptables -I FORWARD -s 10.1.0.0/16 -p tcp --dport 22 -j ACCEPT
    iptables -I FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
    
    一些注意点：
    1、当测试网络防火墙时，默认前提为网络已经正确配置。
    2、当测试网络防火墙时，如果出现问题，请先确定主机防火墙规则的配置没有问题



> [返回目录](#目录)


## 8  
## 动作总结之一
    
    其实，"动作"与"匹配条件"一样，也有"基础"与"扩展"之分。
    同样，使用扩展动作也需要借助扩展模块，但是，扩展动作可以直接使用，不用像使用"扩展匹配条件"那样指定特定的模块。

### 8.1   
### 动作REJECT
    
    REJECT动作的常用选项为--reject-with
    使用--reject-with选项，可以设置提示信息，当对方被拒绝时，会提示对方为什么被拒绝。
    可用值如下
        icmp-net-unreachable
        icmp-host-unreachable
        icmp-port-unreachable,
        icmp-proto-unreachable
        icmp-net-prohibited
        icmp-host-pro-hibited
        icmp-admin-prohibited
    当不设置任何值时，默认值为icmp-port-unreachable。
    
    #示例
    iptables -I INPUT 2 -j REJECT --reject-with icmp-host-unreachable

### 8.2    
### 动作LOG
    
    使用LOG动作，可以将符合条件的报文的相关信息记录到日志中，但当前报文具体是被"接受"，还是被"拒绝"，都由后面的规则控制，换句话说，LOG动作只负责记录匹配到的报
文的相关信息，不负责对报文的其他处理，如果想要对报文进行进一步的处理，可以在之后设置具体规则，进行进一步的处理。
    
    #示例
    iptables -I INPUT -p tcp --dport 22 -j LOG
    
    LOG动作会将报文的相关信息记录在/var/log/message文件中，当然，我们也可以将相关信息记录在指定的文件中，以防止iptables的相关信
    息与其他日志信息相混淆，修改/etc/rsyslog.conf文件（或者/etc/syslog.conf），在rsyslog配置文件中添加如下配置即可。
    
    #vim /etc/rsyslog.conf
    kern.warning /var/log/iptables.log
    加入上述配置后，报文的相关信息将会被记录到/var/log/iptables.log文件中。
    
    完成上述配置后，重启rsyslog服务（或者syslogd）
    #service rsyslog restart
    服务重启后，配置即可生效，匹配到的报文的相关信息将被记录到指定的文件中。
    
    LOG动作也有自己的选项，常用选项如下（先列出概念，后面有示例）
    --log-level选项可以指定记录日志的日志级别，可用级别有emerg，alert，crit，error，warning，notice，info，debug。
    --log-prefix选项可以给记录到的相关信息添加"标签"之类的信息，以便区分各种记录到的报文信息，方便在分析时进行过滤。
    注：--log-prefix对应的值不能超过29个字符。
    
    #示例
    iptables -I INPUT -p tcp --dport 22 -m state --state NEW -j LOG --log-prefix "want-in-from-port-22"



> [返回目录](#目录)


## 9   
## 动作总结之二
    
    SNAT、DNAT、MASQUERADE、REDIRECT
    
    如果想要NAT功能能够正常使用，需要开启Linux主机的核心转发功能。

### 9.1    
### SNAT相关操作
    
    配置SNAT，可以隐藏网内主机的IP地址，也可以共享公网IP，访问互联网，如果只是共享IP的话，只配置如下SNAT规则即可。
    iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -j SNAT --to-source 公网IP
    
    如果公网IP是动态获取的，不是固定的，则可以使用MASQUERADE进行动态的SNAT操作，如下命令表示将10.1网段的报文的源IP修改为eth0网卡中可用的地址。
    iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -o eth0 -j MASQUERADE

### 9.2     
### DNAT相关操作
    
    配置DNAT，可以通过公网IP访问局域网内的服务。
    
    注：理论上来说，只要配置DNAT规则，不需要对应的SNAT规则即可达到DNAT效果。
    但是在测试DNAT时，对应SNAT规则也需要配置，才能正常DNAT，可以先尝试只配置DNAT规则，如果无法正常DNAT，再尝试添加对应的SNAT规则，SNAT规则配置一条即
    可，DNAT规则需要根据实际情况配置不同的DNAT规则。
    
    iptables -t nat -I PREROUTING -d 公网IP -p tcp --dport 公网端口 -j DNAT --to-destination 私网IP:端口号
    iptables -t nat -I PREROUTING -d 公网IP -p tcp --dport 8080 -j DNAT --to-destination 10.1.0.1:80
    iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -j SNAT --to-source 公网IP

### 9.3    
### REDIRECT动作
    
    在本机进行目标端口映射时可以使用REDIRECT动作。
    iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
    配置完成上述规则后，其他机器访问本机的80端口时，会被映射到8080端口



> [返回目录](#目录)


----

## 10
## 常用套路
    
    1、规则的顺序非常重要。
    如果报文已经被前面的规则匹配到，iptables则会对报文执行对应的动作，通常是ACCEPT或者REJECT，报文被放行或拒绝以后，即使后面的规则也能匹配到刚才放行或拒绝的报 文，也没有机会再对报文执行相应的动作了（前面规则的动作为LOG时除外），所以，针对相同服务的规则，更严格的规则应该放在前面。
    
    2、当规则中有多个匹配条件时，条件之间默认存在"与"的关系。
    如果一条规则中包含了多个匹配条件，那么报文必须同时满足这个规则中的所有匹配条件，报文才能被这条规则匹配到。
    
    3、在不考虑1的情况下，应该将更容易被匹配到的规则放置在前面。
    比如，你写了两条规则，一条针对sshd服务，一条针对web服务。 假设，一天之内，有20000个请求访问web服务，有200个请求访问sshd服务， 那么，应该将针对web服务的规则放在前面，针对sshd的规则放在后面，因为访问web服务的请求频率更高。 如果将sshd的规则放在前面，当报文是访问web服务时，sshd的规则也要白白的验证一遍，由于访问web服务的频率更高，白白耗费的资源就更多。 如果web服务的规则放在前面，由于访问web服务的频率更高，所以无用功会比较少。 换句话说就是，在没有顺序要求的情况下，不同类别的规则，被匹配次数多的、匹配频率高的规则应该放在前面。
    
    4、当iptables所在主机作为网络防火墙时，在配置规则时，应着重考虑方向性，双向都要考虑，从外到内，从内到外。
    
    5、在配置iptables白名单时，往往会将链的默认策略设置为ACCEPT，通过在链的最后设置REJECT规则实现白名单机制，而不是将链的默认策略设置为DROP，如果将链的默认
    策略设置为DROP，当链中的规则被清空时，管理员的请求也将会被DROP掉。
    


> [返回目录](#目录)






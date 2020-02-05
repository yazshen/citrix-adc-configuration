# Citrix ADC转发真实客户端IP地址

Citrix ADC 是一款应用交付和负载均衡解决方案，无论您的 Web、传统和云原生应用托管于何处，均可提供高质量的用户体验。
Load Balancing是Citrix ADC的核心功能，客户端连接到Citrix ADC后，Citrix ADC通过Subnet IP(SNIP)作为源地址向后端服务器发起连接。

很多时候，后端服务器都需要记录真实客户端IP地址（溯源）。Citrix ADC可以通过不同的方式，在HTTP/TCP等应用上灵活得帮助用户实现这个需求。

## HTTP应用溯源功能
RFC 7239（Forwarded HTTP Extension）标准规定使用HTTP协议的扩展头X-Forwarded-For来表示HTTP请求端真实IP地址。

Citrix负载均衡设备可以在HTTP服务上配置X-Forwarded-For字段并写入真实客户端IP地址。HTTP应用服务器可以通过读取X-Forwarded-For字段记录来溯源。

### 配置方法
编辑Load Balancing Service，启用Client IP并配置头名称为X-Forwarded-For
![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-01.png)

### IPv4环境验证

    客户端：192.168.202.200
    VIP服务器： 192.168.202.26
    SNIP接口：192.168.202.23
    服务器： 192.168.202.201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-02.png)

### IPv6环境验证

    客户端：fd00::192:168:202:200
    VIP服务器： fd00::192:168:202:26
    SNIP接口：fd00::192:168:202:23
    服务器：fd00::192:168:202:201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-03.png)

## TCP应用溯源功能（ProxyProtocol）
Proxy Protocol是HAProxy于2010年开发和设计的一个Internet协议，通过为tcp添加一个很小的头信息，来方便的传递客户端信息（协议栈、源IP、目的IP、源端口、目的端口等)，在网络情况复杂又需要获取用户真实IP时非常有用。其本质是在三次握手结束后由代理在连接中插入了一个携带了原始连接四元组信息的数据包。

参考链接：https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt

Citrix负载均衡通过rewrite脚本，可以在TCP请求的payload中插入proxy protocol规范信息，真实服务器可以通过proxy protocol规范来读取真实客户端IP地址。

### 配置方法
创建Rewrite Action

    add rewrite action proxyprotocol insert_before "client.tcp.payload(1)" "\"PROXY TCP4 \" + client.ip.src + \" \" + client.ip.dst + \" \" + client.tcp.srcport + \" \" + client.tcp.dstport +\"\r\n\""

    add rewrite action proxyprotocolV6 insert_before "client.tcp.payload(1)" "\"PROXY TCP6 \" + client.ipv6.src + \" \" + client.ipv6.dst + \" \" + client.tcp.srcport + \" \" + client.tcp.dstport +\"\r\n\""

创建Rewrite Policy

    add rewrite policy tcpsourceip-22 "CLIENT.TCP.DSTPORT.EQ(22)" proxyprotocol
    
    add rewrite policy tcpsourceipV6-22 "CLIENT.TCP.DSTPORT.EQ(22)" proxyprotocolV6
    
绑定Rewrite Policy到TCP VIP

    bind lb vserver 192.168.202.26_tcp_22 -policyName tcpsourceip-22 -priority 100 -gotoPriorityExpression NEXT -type REQUEST
    
    bind lb vserver fd00::192:168:202:26_tcp_22 -policyName tcpsourceipV6-22 -priority 100 -gotoPriorityExpression NEXT -type REQUEST

### IPv4环境验证

    客户端：192.168.202.200
    VIP服务器： 192.168.202.26
    SNIP接口：192.168.202.23
    服务器： 192.168.202.201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-04.png)

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-05.png)

### IPv6环境验证

    客户端：fd00::192:168:202:200
    VIP服务器： fd00::192:168:202:26
    SNIP接口：fd00::192:168:202:23
    服务器：fd00::192:168:202:201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-06.png)

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-07.png)

## TCP应用溯源功能（TCP Option）
从Citrix ADC 13.0版本开始，支持在TCP Option字段插入客户端IP地址信息。这样的方法是通过在第一个数据包的TCP选项中发送客户端IP地址。如果后端服务器使用TCP选项读取客户端IP地址，则设备将使用TCP配置文件中的TCP选项号。 IP地址存于TCP选项号28（不限于28，可在TCP配置上定义）。 TCP选项方法在将客户端IP地址传送到后端服务器时包括插入和转发功能。在TCP选项配置中，设备添加了一个TCP选项28，以插入客户端IP地址并将其转发到后端服务器。

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-08.png)

注：如果启用TCP Option插入客户端IP地址，那么连接多路复用(Connection Multiplexing)功能将被禁止。

### 配置方法
测试版本：Citrix ADC 13.0 47.22

进入System -> Profiles -> TCP Profile，创建一个新的TCP Profile。勾选“Client IP TCP Option”选项并定义Option Number，这里我们选择“28”

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-09.png)

进入Traffic Management -> Load Balancing -> Services，编辑相关Service。点击Profiles，选择TCP Profile为刚才创建的Profile。

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-10.png)

### IPv4环境验证

    客户端：192.168.202.200
    VIP服务器： 192.168.202.15
    SNIP接口：192.168.202.13
    服务器： 192.168.202.201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-11.png)

### IPv6环境验证

    客户端：fd00::192:168:202:200
    VIP服务器： fd00::192:168:202:15
    SNIP接口：fd00::192:168:202:13
    服务器：fd00::192:168:202:201

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-12.png)

## Citrix负载均衡Syslog溯源
除了可以把真实客户端IP地址转发给后端服务器，Citrix ADC自身还可以通过Syslog方式记录真实客户端IP地址信息。用户也可以通过Syslog发送到收集器并记录溯源。

### 配置方法
进入System -> Auditing -> Syslog，创建新的Syslog收集器

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-13.png)

创建Syslog Policy

![forward-client-ip](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/forward-client-ip-14.png)

### IPv4环境验证

    客户端：192.168.202.200
    VIP服务器： 192.168.202.26
    SNIP接口：192.168.202.23
    服务器： 192.168.202.201

    2020-01-20 15:28:08	Local0.Info	192.168.201.21	 2020/01/20:15:20:58  DC-ADC21 0-PPE-0 : TCP CONN_DELINK 1778 0 :  Source 192.168.202.200:54120 - Vserver 192.168.202.26:80 - NatIP 192.168.202.23:2500 - Destination 192.168.202.201:80 - Delink Time 2020/01/20:15:20:58  - Total_bytes_send 78 - Total_bytes_recv 355

### IPv6环境验证

    客户端：fd00::192:168:202:200
    VIP服务器： fd00::192:168:202:26
    SNIP接口：fd00::192:168:202:23
    服务器：fd00::192:168:202:201

    2020-01-20 15:39:00	Local0.Info	192.168.201.21	 2020/01/20:15:31:50  DC-ADC21 0-PPE-0 : TCP CONN_DELINK 2193 0 :  Source fd00::192:168:202:200:41860 - Vserver fd00::192:168:202:26:80 - NatIP fd00::192:168:202:23:2209 - Destination fd00::192:168:202:201:80 - Delink Time 2020/01/20:15:31:50  - Total_bytes_send 86 - Total_bytes_recv 355

### NAT64环境验证

    客户端：fd00::192:168:202:200
    VIP服务器： fd00::192:168:202:26
    SNIP接口：192.168.202.23
    服务器：192.168.202.201

    2020-01-21 13:41:52	Local0.Info	192.168.201.21	 2020/01/21:13:41:51  DC-ADC21 0-PPE-0 : TCP CONN_DELINK 5741 0 :  Source fd00::192:168:202:200:43460 - Vserver fd00::192:168:202:26:80 - NatIP 192.168.202.23:10058 - Destination 192.168.202.201:80 - Delink Time 2020/01/21:13:41:51  - Total_bytes_send 86 - Total_bytes_recv 355



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


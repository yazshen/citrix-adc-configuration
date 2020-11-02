# TACACS外部认证配置(Cisco ISE)

Citrix ADC不仅支持内置认证的用户系统，同时也支持通过外部认证方式管理Citrix ADC设备。其中外部认证包括：TACACS和Radius。

以下是通过Cisco ISE实现外部用户认证登录Citrix ADC设备：
+ Cisco ISE版本：2.4.0.357
+ Citrix ADC版本：12.1 Build 59-16

## 参考文档
Citrix ADC: https://support.citrix.com/article/CTX113820

Cisco ISE: https://www.cisco.com/c/en/us/td/docs/security/ise/2-4/admin_guide/b_ISE_admin_guide_24/m_ise_tacacs_device_admin.html

## Citrix ADC配置
### 创建TACACS Server配置
选择菜单：Configuration -> System -> Authentication -> Basic Policies -> TACACS -> Servers，创建一个新的Server
+ 点击"Test Connection"测试TACACS服务器连接
+ 配置Authorization为"On"并

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-01.png)

### 创建TACACS Policy配置
选择菜单：Configuration -> System -> Authentication -> Advanced Policies -> Policy，创建一个新的Policy
+ 选择刚才创建的TACACS Server
+ Expression配置为"true"

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-02.png)

### 绑定TACACS Policy
选择菜单：Configuration -> System -> Authentication -> Advanced Policies -> Policy，点击Global Bindings
+ 选择刚才创建的TACACS Policy
+ 输入Priority

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-03.png)

## Cisco ISE配置

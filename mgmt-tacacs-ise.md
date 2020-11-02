# TACACS外部认证配置(Cisco ISE)

Citrix ADC不仅支持内置认证的用户系统，同时也支持通过外部认证方式管理Citrix ADC设备。其中外部认证包括：TACACS和Radius。

以下是通过Cisco ISE实现外部用户admin权限认证并登录Citrix ADC设备：
+ Cisco ISE版本：2.4.0.357
+ Citrix ADC版本：12.1 Build 59-16

## Citrix ADC配置
### 创建TACACS Server
选择菜单：Configuration -> System -> Authentication -> Basic Policies -> TACACS -> Servers，创建一个新的Server
+ 点击"Test Connection"测试TACACS服务器连接
+ 配置Authorization为"On"并

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-01.png)

### 创建TACACS Policy
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
### 创建User Identity Group
选择菜单：Worker Center -> Network Access -> id Groups -> User Identity Groups, 创建一个新的Group
+ 输入Group名称，本篇文章我们会使用Group名称来关联Citrix ADC认证

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-04.png)

### 创建Network Access User
选择菜单：Worker Center -> Network Access -> Network Access Users, 创建一个新的User
+ 输入用户名、密码
+ 绑定User Group，选择刚才创建的Group

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-05.png)

### 创建Network Resource Device
选择菜单：Worker Center -> Network Access -> Network Resources -> Network Devices, 创建一个新的Device
+ 输入设备名称、IP地址
+ 启用TACACS认证并定义TACACS Key

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-06.png)

### 创建TACACS Command Set
选择菜单：Worker Center -> Device Administration -> Poicy Elements -> Results -> TACACS Command Sets, 创建一个新的Command Set
+ 默认admin权限，所以我们启用"Permit any command that is not listed below"

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-07.png)

### 创建TACACS Profile
选择菜单：Worker Center -> Device Administration -> Poicy Elements -> Results -> TACACS Profiles, 创建一个新的Profile
+ 配置Common Task Type为"Shell"
+ 启用"Default Privilege"并配置为"15"

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-08.png)

### 启用TACACS Profile
选择菜单：Worker Center -> Device Administration -> Device Admin Policy Sets -> Authorization Policy - Global Exceptions，创建一个新的Rule
+ 选择刚才创建的TACACS Command Set
+ 选择刚才创建的TACACS Profile

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-09.png)

## 排障
SSH登录Citrix ADC设备命令行，然后输入如下命令检查外部用户认证过程：
+ shell
+ more /tmp/aaad.debug

![mgmt-tacacs-ise](https://github.com/yazshen/citrix-adc-configuration/blob/master/images/mgmt-tacacs-ise-10.png)

## 参考文档
Citrix ADC: https://support.citrix.com/article/CTX113820

Cisco ISE: https://www.cisco.com/c/en/us/td/docs/security/ise/2-4/admin_guide/b_ISE_admin_guide_24/m_ise_tacacs_device_admin.html



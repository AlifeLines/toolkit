[comment]: <> (![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1517415003170&di=5df719d3755d8b03220866409599a5e6&imgtype=0&src=http%3A%2F%2Fimg001.21cnimg.com%2Fphotos%2Falbum%2F20170123%2Fm600%2FBD39878FA67A1C90400FE84257E09EA6.png))
ubuntu安装vpn

# 安装说明

用法很简单： 总结成一句话就是：除了类型要选对以外，其他的一路回车就好了23333

# 特性

  * 服务端要求：Ubuntu或者CentOS-6/7或者Debian
  * 客户端：
  * iOS/OSX=>ikev1,ikev2
  * Andriod=>ikev1
  * WindowsPhone=>ikev2
  * 其他Windows平台=>ikev2
  * 可使用自己的私钥和根证书，也可自动生成
  * 证书可绑定域名或ip
  * 要是图方便可一路回车

### 下载脚本

    
    
        wget --no-check-certificate https://raw.githubusercontent.com/quericy/one-key-ikev2-vpn/master/one-key-ikev2.sh
    

### 运行脚本

    
    
    chmod +x one-key-ikev2.sh
    bash one-key-ikev2.sh
    

### 自动配置

等待自动配置部分内容后，选择vps类型（OpenVZ还是Xen、KVM），选错将无法成功连接，请务必核实服务器的类型。输入服务器ip或者绑定的域名(连接vpn时服务器地址将需要与此保持一致,如果是导入泛域名证书这里需要写*.域名的形式)；

### 证书

选择使用使用证书颁发机构签发的SSL证书还是生成自签名证书

  * 如果选择no,使用自签名证书（客户端如果使用IkeV2方式连接，将需要导入生成的证书并信任）则需要填写证书的相关信息(C,O,CN)，为空将使用默认值(default value)，确认无误后按任意键继续,后续安装过程中会出现输入两次pkcs12证书的密码的提示(可以设置为空)
  * 如果选择yes，使用SSL证书（如果证书是被信任的，后续步骤客户端将无需导入证书）请在继续下一步之前，将以下文件按提示命名并放在脚本相同的目录下（SSL证书详细配置和自动续期方案可见https://quericy.me/blog/860/ ）：

  * ca.cert.pem 证书颁发机构的CA，比如Let‘s Encrypt的证书,或者其他链证书；

  * server.cert.pem 签发的域名证书；
  * server.pem 签发域名证书时用的私钥；

### 是否使用SNAT规则(可选)

默认为不使用.使用前请确保服务器具有不变的静态公网ip,可提升防火墙对数据包的处理速度.如果服务器网络设置了NAT(如AWS的弹性ip机制),则填写网卡连接接口的ip地址(参见KinonC提供的方案:#36).

### 防火墙配置

默认配置iptables(如果使用的是firewall(如CentOS7)请选择yes自动配置firewall,将无视SNAT并跳过后续的补充网卡接口步骤).补充网卡接口信息,为空则使用默认值(Xen、KVM默认使用eth0,OpenVZ默认使用venet0).如果服务器使用其他公网接口需要在此指定接口名称,填写错误VPN连接后将无法访问外网)

### 看到install Complete字样即表示安装完成

默认用户名密码将以黄字显示，可根据提示自行修改配置文件中的用户名密码,多用户则在配置文件中按格式一行一个(多用户时用户名不能使用%any),保存并重启服务生效。

将提示信息中的证书文件ca.cert.pem拷贝到客户端，修改后缀名为.cer后导入。ios设备使用Ikev1无需导入证书，而是需要在连接时输入共享密钥，共享密钥即是提示信息中的黄字PSK.

# 客户端配置说明

  * 连接的服务器地址和证书保持一致,即取决于签发证书ca.cert.pem时使用的是ip还是域名;

  * Android/iOS/OSX 可使用ikeV1,认证方式为用户名+密码+预共享密钥(PSK);

  * iOS/OSX/Windows7+/WindowsPhone8.1+/Linux 均可使用IkeV2,认证方式为用户名+密码。使用SSL证书则无需导入证书；使用自签名证书则需要先导入证书才能连接,可将ca.cert.pem更改后缀名作为邮件附件发送给客户端,手机端也可通过浏览器导入,其中:

  * iOS/OSX 的远程ID和服务器地址保持一致,用户鉴定选择”用户名”.如果通过浏览器导入,将证书放在可访问的远程外链上,并在系统浏览器(Safari)中访问外链地址;

  * Windows PC 系统导入证书需要导入到“本地计算机”的”受信任的根证书颁发机构”,以”当前用户”的导入方式是无效的.推荐运行mmc添加本地计算机的证书管理单元来操作;
  * WindowsPhone8.1 登录时的用户名需要带上域信息,即wp”关于”页面的设备名称\用户名,也可以使用%any %any : EAP “密码”进行任意用户名登录,但指定了就不能添加其他用户名了.
  * WindowsPhone10 的vpn还存在bug(截至10586.164),ikeV2方式可连接但系统流量不会走vpn,只能等微软解决. (截至14393.5 ,此bug已经得到修复,现在WP10已经可以正常使用IkeV2.)
  * Windows10 也存在此bug,部分Win10系统连接后ip不变,没有自动添加路由表,使用以下方法可解决(本方法由 bigbigfish 童鞋提供):
    * 手动关闭vpn的split tunneling功能(在远程网络上使用默认网关);
    * 也可使用powershell修改,进入CMD窗口,运行如下命令:

    
    
    powershell    #进入ps控制台
    get-vpnconnection    #检查vpn连接的设置（包括vpn连接的名称）
    set-vpnconnection "vpn连接名称" -splittunneling $false    #关闭split tunneling
    get-vpnconnection   #检查修改结果
    exit   #退出ps控制台
    

# 卸载方式

  * 进入脚本所在目录的strongswan文件夹执行`make uninstall`

  * 删除脚本所在目录的相关文件(one-key-ikev2.sh,strongswan.tar.gz,strongswan文件夹,my_key文件夹).

  * 卸载后记得检查iptables配置.

参考资料

[CentOS/Ubuntu一键安装IPSEC/IKEV2 VPN服务器](https://quericy.me/blog/699/)


[comment]: <tags> (vpn,ubuntu)
[comment]: <description> (安装vpn)
[comment]: <title> (ubuntu安装vpn)
[comment]: <author> (夏洛之枫)
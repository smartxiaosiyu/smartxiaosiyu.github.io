---
title: 'iOS 签名机制'
date: 2021-12-05 16:12:26
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### 加壳
从AppStore下载的App，会被加密
利用特殊的算法，对我们可执行文件的编码进行改变

壳程序里面有（可执行文件（已加密））———> 执行到———>内存中（壳程序里面有（可执行文件（已加密）））———>解密 ———>内存中（壳程序里面有（可执行文件（解密）））

脱壳 
硬脱壳： 通过解密算法 从壳程序里解出可执行文件
动态脱壳：当壳程序执行到内存中解密出可执行文件 ，我们从内存中拿到解密后的可执行文件

### iOS签名机制
加密方法有两个
+ 对称密码 加密和解密用的同一个密钥
+  非对称密码（公钥密码）加密和解密用的密钥是不一样的
密钥 56bit

### 对称密码：
**DES ** 只能加密64bit的明文 已经被破解
**3DES** 3DES，将DES重复3次所得到的一种密码算法，也叫做3重DES 目前还被一些银行等机构使用，但处理速度不高，安全性逐渐暴露出问题
**AES** 取代DES成为新标准的一种对称密码算法 它经过了全世界密码学家所进行的高品质验证工作

### 如何解决密钥配送问题
有以下几种解决密钥配送的方法
+ 事先共享密钥
+ 密钥分配中心
+ Diffie-Hellman密钥交换

### 公钥密码
公钥密码中，密钥分为加密密钥、解密密钥2种，它们并不是同一个密钥
加密密钥，一般是公开的，因此该密钥称为公钥（public key）
解密密钥，由消息接收者自己保管的，不能公开，因此也称为私钥（private key）
公钥和私钥是一 一对应的，是不能单独生成的，一对公钥和密钥统称为密钥对（key pair）
由公钥加密的密文，必须使用与该公钥对应的私钥才能解密
由私钥加密的密文，必须使用与该私钥对应的公钥才能解密
![](https://smartxiaosiyu.github.io/post-images/1638709484493.png)
目前使用最广泛的公钥密码算法是RSA

### 解决密钥配送问题
由消息的接收者，生成一对公钥、私钥
将公钥发给消息的发送者
消息的发送者使用公钥加密消息

对称密码的缺点
不能很好地解决密钥配送问题

公钥密码的缺点
加密解密速度比较慢

混合密码系统，是将对称密码和公钥密码的优势相结合的方法
解决了公钥密码速度慢的问题
并通过公钥密码解决了对称密码的密钥配送问题

网络上的密码通信所用的SSL/TLS都运用了混合密码系统

### 混合密码-加密
![](https://smartxiaosiyu.github.io/post-images/1638710695280.png)
![](https://smartxiaosiyu.github.io/post-images/1638710701498.png)

### 单向散列函数
单向散列函数，可以根据根据消息内容计算出散列值
散列值的长度和消息的长度无关，无论消息是1bit、10M、100G，单向散列函数都会计算出固定长度的散列值
+ 根据任意长度的消息，计算出固定长度的散列值
+ 计算速度快，能快速计算出散列值
+ 消息不同，散列值也不同
+ 具备单向性

### 数字签名
![](https://smartxiaosiyu.github.io/post-images/1638873958154.png)
数字签名不能保证机密性？
数字签名的作用不是为了保证机密性，仅仅是为了能够识别内容有没有被篡改

数字签名的作用
确认消息的完整性
识别消息是否被篡改
防止消息发送人否认

### 数字签名无法解决的问题
要正确使用签名，前提是
用于验证签名的公钥必须属于真正的发送者

如果遭遇了中间人攻击，那么
公钥将是伪造的
数字签名将失效

所以在验证签名之前，首先得先验证公钥的合法性

如何验证公钥的合法性？
证书
![](https://smartxiaosiyu.github.io/post-images/1638876813483.png)
密码学中的证书，全称叫公钥证书（Public-key Certificate，PKC），跟驾驶证类似
里面有姓名、邮箱等个人信息，以及此人的公钥
并由认证机构（Certificate Authority，CA）施加数字签名

CA就是能够认定“公钥确实属于此人”并能够生成数字签名的个人或者组织
![](https://smartxiaosiyu.github.io/post-images/1638878379794.png)

![](https://smartxiaosiyu.github.io/post-images/1638878575199.png)

### iOS签名机制 – 流程图
![](https://smartxiaosiyu.github.io/post-images/1638881402683.png)
安装
+ 1.Mac私钥签名App 生成  -》（App和App内容签名）
+ 2.App私钥签名 Mac公钥 生成  -》（Mac公钥和Mac公钥签名）
+ 3.App私钥签名 （Mac公钥和Mac公钥签名）+ Devices + app id + entitlements 生成 -》 （（Mac公钥和Mac公钥签名）+ Devices + app id + entitlements 和 签名）
验证
+ App公钥验证安装3的签名 和 安装3的内容验证
+ 核实Devices + app id + entitlements
+ App公钥验证安装2的签名获取 Mac公钥
+ Mac公钥 验证安装1的签名 核实App内容

签名机制就是为了验证App不被人篡改

CertificateSigningRequest.certSigningRequest文件
就是Mac设备的公钥

ios_development.cer、ios_distribution.cer文件
利用Apple后台的私钥，对Mac设备的公钥进行签名后的证书文件  安装2

生成mobileprovision 选择设备、证书  安装3

![](https://smartxiaosiyu.github.io/post-images/1638883511070.png)

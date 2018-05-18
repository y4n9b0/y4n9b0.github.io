---
layout: post
title:  OpenSSL CA
date:   2018-05-09 09:00:00 +0800
categories: Linux
tags: OpenSSL CA
published: true
---

* content
{:toc}

# PKI
https://blog.csdn.net/evsqiezi/article/details/43411937  
TODO

# CA
CA: Certificate Authority  
证书基本的文件类型和协议有: PEM、DER、PFX、JKS、KDB、CER、KEY、CSR、CRT、CRL 、OCSP、SCEP等。

* PEM – Openssl使用 PEM(Privacy Enhanced Mail)格式来存放各种信息,它是 openssl 默认採用的信息存放方式。
  Openssl 中的 PEM 文件一般包括例如以下信息:
  * 内容类型:表明本文件存放的是什么信息内容,它的形式为“——-BEGIN XXXX ——”,与结尾的“——END XXXX——”相应。
  * 头信息:表明数据是假设被处理后存放,openssl 中用的最多的是加密信息,比方加密算法以及初始化向量 iv。
  * 信息体:为 BASE64 编码的数据。
    能够包含全部私钥（RSA 和 DSA）、公钥（RSA 和 DSA）和 (x509) 证书。它存储用 Base64 编码的 DER 格式数据，用 ascii 报头包围，因此适合系统之间的文本模式传输。

  使用PEM格式存储的证书：
    ```
    —–BEGIN CERTIFICATE—–
    MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
    ………
    1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
    —–END CERTIFICATE—–
    ```

  使用PEM格式存储的私钥：
    ```
    —–BEGIN RSA PRIVATE KEY—–
    MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
    ………
    1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
    —–END RSA PRIVATE KEY—–
    ```

  使用PEM格式存储的证书请求文件：
    ```
    —–BEGIN CERTIFICATE REQUEST—–
    MIICJjCCAdCgAwIBAgIBITANBgkqhkiG9w0BAQQFADCBqTELMAkGA1UEBhMCVVMx
    ………
    1p8h5vkHVbMu1frD1UgGnPlOO/K7Ig/KrsU=
    —–END CERTIFICATE REQUEST—–
    ```

* DER – 辨别编码规则 (DER) 可包括全部私钥、公钥和证书。它是大多数浏览器的缺省格式，并按 ASN1 DER 格式存储。它是无报头的 － PEM 是用文本报头包围的 DER。  

* PFX 或 P12 – 公钥加密标准 #12 (PKCS#12) 可包括全部私钥、公钥和证书。  
  其以二进制格式存储，也称为 PFX 文件。  

  通常能够将Apache/OpenSSL使用的“KEY文件 + CRT文件”格式合并转换为标准的PFX文件，你能够将PFX文件格式导入到微软IIS 5/6、微软ISA、微软Exchange Server等软件。转换时须要输入PFX文件的加密password。

* JKS – 通常能够将Apache/OpenSSL使用的“KEY文件 + CRT文件”格式”转换为标准的Java Key Store(JKS)文件。  
  JKS文件格式被广泛的应用在基于JAVA的WEBserver、应用server、中间件。  
  你能够将JKS文件导入到TOMCAT、 WEBLOGIC 等软件。  

* KDB – 通常能够将Apache/OpenSSL使用的“KEY文件 + CRT文件”格式转换为标准的IBM KDB文件。  
  KDB文件格式被广泛的应用在IBM的WEBserver、应用server、中间件。你能够将KDB文件导入到IBM HTTP Server、IBM Websphere 等软件。  

* CSR － 证书请求文件(Certificate Signing Request)。  
  生成 X509 数字证书前,一般先由用户提交证书申请文件,然后由 CA 来签发证书。大致步骤例如以下(X509 证书申请的格式标准为 pkcs#10 和 rfc2314):  
  * 用户生成自己的公私钥对;
  * 构造自己的证书申请文件,符合 PKCS#10 标准。该文件主要包括了用户信息、公钥以及一些可选的属性信息,并用自己的私钥给该内容签名;
  * 用户将证书申请文件提交给 CA;
  * CA 验证签名,提取用户信息,并加上其它信息(比方颁发者等信息),用 CA 的私钥签发数字证书;  
  **Note**:  
  数字证书(如x.509)是将用户(或其它实体)身份与公钥绑定的信息载体。  
  一个合法的数字证书不仅要符合 X509 格式规范,还必须有 CA的签名。用户不仅有自己的数字证书,还必须有相应的私钥。X509v3数字证书主要包括的内容有:证书版本号、证书序列号、签名算法、颁发者信息、有效时间、持有者信息、公钥信息、颁发者 ID、持有者 ID 和扩展项。  


* OCSP– 在线证书状态协议(OCSP,Online Certificate StatusProtocol,rfc2560)用于实时表明证书状态。  
  OCSP client通过查询 OCSP服务来确定一个证书的状态,能够提供给使用者一个或多个数字证书的有效性资料，它建立一个可实时响应的机制。让用户能够实时确认每一张证书的有效性，解决由CRL引发的安全问题。  
  OCSP 能够通过 HTTP协议来实现。  
  rfc2560 定义了 OCSP client和服务端的消息格式。

* CER  － 一般指使用DER格式的证书。

* CRT － 证书文件。能够是PEM格式。

* KEY   － 一般指PEM格式的私钥文件。

* CRL－ 证书吊销列表 (Certification Revocation List) 是一种包括撤销的证书列表的签名数据结构。CRL是证书撤销状态的发布形式,CRL 就像信用卡的黑名单,用于发布某些数字证书不再有效。CRL是一种离线的证书状态信息。  

  它以一定的周期进行更新。CRL 能够分为全然 CRL和增量 CRL。在全然 CRL中包括了全部的被撤销证书信息,增量 CRL 由一系列的 CRL 来表明被撤销的证书信息,它每次发布的 CRL 是对前面公布 CRL的增量扩充。主要的 CRL 信息有:被撤销证书序列号、撤销时间、撤销原因、签名者以及 CRL 签名等信息。基于 CRL的验证是一种不严格的证书认证。CRL 能证明在 CRL 中被撤销的证书是无效的。  

  可是,它不能给出不在 CRL中的证书的状态。假设运行严格的认证,须要採用在线方式进行认证,即 OCSP认证。通常是由CA签名的一组电子文档，包含了被废除证书的唯一标识（证书序列号）。CRL用来列出已经过期或废除的数字证书。它每隔一段时间就会更新，因此必须定期下载该清单。才会取得最新信息。

* SCEP － 简单证书注冊协议。

  基于文件的证书登记方式须要从您的本地计算机将文本文件复制和粘贴到证书公布中心。和从证书公布中心复制和粘贴到您的本地计算机。 SCEP能够自己主动处理这个过程可是CRLs仍然须要手工的在本地计算机和CA公布中心之间进行复制和粘贴。

* PKCS7 – 加密消息语法(pkcs7),是各种消息存放的格式标准。这些消息包含:数据、签名数据、数字信封、签名数字信封、摘要数据和加密数据。

* PKCS12 – pkcs12 (个人数字证书标准)用于存放用户证书、crl、用户私钥以及证书链。pkcs12 中的私钥是加密存放的。

# 数字证书原理
https://www.zhihu.com/question/24294477  
TODO

# OpenSSL
  https://blog.csdn.net/scuyxi/article/details/54884976  
  man openssl  
  TODO

# Create CA
知名免费CA证书   
https://www.zhihu.com/question/22520304/answer/137070944  
TODO

1.  建立CA目录结构  
    按照 OpenSSL 的默认配置建立 CA ，需要在文件系统中建立相应的目录结构。相关的配置内容一般位于 /usr/lib/ssl/openssl.cnf 内，安装OpenSSL后，系统会在/etc/ssl/目录下生成openssl.cnf文件（不同的操作系统可能位于不同的目录，本文所用系统为 ubuntu 16.04）。
    ```
    $ cd /etc/pki
    $ mkdir -p ./demoCA/{private,newcerts}
    $ touch ./demoCA/index.txt
    $ echo 01 > ./demoCA/serial
    ```
    产生的目录结构如下：
    ```
    demoCA/
    ├── index.txt
    ├── newcerts
    ├── private
    └── serial
    ```

2.  生成 CA 证书的私钥  
    ```
    $ openssl genrsa -aes256 -out ./demoCA/private/cakey.pem 4096
    $ chmod 400 ./demoCA/private/cakey.pem
    ```
    **参数解释:**  
    * genrsa  
      用于生成 RSA 密钥对的 OpenSSL 命令。

    * -aes256  
      使用 aes256 位对称加密算法加密私钥，该参数需要用户在私钥生成过程中输入一个口令用于加密。今后使用该私钥时，需要输入相应的口令。如果不加该选项，则不对私钥进行加密。

    * -out ./demoCA/private/cakey.pem  
      令生成的私钥保存到文件 ./demoCA/private/cakey.pem 。

    * 4096  
      RSA 模数位数，在一定程度上表征了私钥强度。  

    该命令输出如下，用户应输入自己的私钥口令并确认：
    ```
    Generating RSA private key, 4096 bit long modulus
    ...++
    ............................................................................................................................................................++
    e is 65537 (0x10001)
    Enter pass phrase for cakey.pem:<enter your pass-phrase>
    Verifying - Enter pass phrase for cakey.pem:<re-enter your pass-phrase>
    ```

3.  生成 CA 证书请求  
    为了获取一个 CA 根证书，我们需要先制作一份证书请求。用先前生成的 CA 私钥对证书请求签名。
    ```
    $ openssl req -new -days 365 -key ./demoCA/private/cakey.pem -sha256 -extensions v3_req -out ./demoCA/careq.pem
    ```
    **参数解释:**
    * req  
      用于生成证书请求的 OpenSSL 命令。

    * -new  
      生成一个新的证书请求。该参数将令 OpenSSL 在证书请求生成过程中要求用户填写一些相应的字段。

    * -days 365  
      从生成之时算起，证书时效为 365 天。

    * -key ./demoCA/private/cakey.pem  
      指定 ./demoCA/private/cakey.pem 为证书所使用的私钥文件。

    * -sha256  
      使用SHA-256算法生成消息摘要

    * -extensions v3_req  
      生成v3证书请求，默认是v1,不带扩展属性。  
      具体的 v3_req 设置可以在使用的默认配置文件 `/usr/lib/ssl/openssl.cnf` 里修改（不建议这样操作，建议拷贝一份配置文件 `**.cnf`，命令中添加参数`-config **.cnf` 使用指定配置文件，需要时修改该文件即可）。

    * -out ./demoCA/careq.pem  
      令生成的证书请求保存到文件./demoCA/careq.pem

    该命令将提示用户输入私钥口令并填写证书相关信息字段，输出如下：
    ```
    Enter pass phrase for ./demoCA/private/cakey.pem:<enter your pass-phrase>
    You are about to be asked to enter information that will be incorporated into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:CN
    State or Province Name (full name) [Some-State]:Sichuan
    Locality Name (eg, city) []:Chengdu
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Phicomm
    Organizational Unit Name (eg, section) []:SmartHome
    Common Name (e.g. server FQDN or YOUR name) []:Bob
    Email Address []:step2hell@qq.com

    Please enter the following 'extra' attributes to be sent with your certificate request
    A challenge password []:<enter another pass-phrase>
    An optional company name []:
    ```
    **Note:**
    * extra challenge password:  
     [https://serverfault.com/questions/266232/what-is-a-challenge-password](https://serverfault.com/questions/266232/what-is-a-challenge-password)

4.  对 CA 证书请求进行签名  
    在实际应用中，用户可以通过向知名 CA 递交证书请求来申请证书。但是在这里，我们需要建立的是一个根 CA ，只能由我们自己来对证书请求进行签名。所以我们让 OpenSSL 使用证书请求中附带的密钥对该请求进行签名，也就是所谓的“ self sign ”：
    ```
    $ openssl ca -selfsign -in ./demoCA/careq.pem -out ./demoCA/cacert.pem
    ```
    **参数解释:**

    * ca  
      用于执行 CA 相关操作的 OpenSSL 命令。

    * -selfsign  
      使用对证书请求进行签名的密钥对来签发证书。

    * -in careq.pem  
      指定 careq.pem 为证书请求文件。

    * -out ./demoCA/cacert.pem  
      指定 ./demoCA/cacert.pem 为输出的证书。

    该命令要求用户输入私钥口令并输出相关证书信息，请求用户确认：
    ```
    Using configuration from /usr/lib/ssl/openssl.cnf
    Enter pass phrase for ./demoCA/private/cakey.pem:
    Check that the request matches the signature
    Signature ok
    Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: May  9 02:51:28 2018 GMT
            Not After : May  9 02:51:28 2019 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = Sichuan
            organizationName          = Phicomm
            organizationalUnitName    = SmartHome
            commonName                = Bob
            emailAddress              = step2hell@qq.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                D8:2C:B7:93:7B:D2:E2:14:AF:8D:D8:A2:5C:17:2D:5C:18:FB:0E:55
            X509v3 Authority Key Identifier:
                keyid:D8:2C:B7:93:7B:D2:E2:14:AF:8D:D8:A2:5C:17:2D:5C:18:FB:0E:55

    Certificate is to be certified until May  9 02:51:28 2019 GMT (365 days)
    Sign the certificate? [y/n]:y

    1 out of 1 certificate requests certified, commit? [y/n]y
    Write out database with 1 new entries
    Data Base Updated
    ```
    至此，我们便已成功建立了一个私有根 CA 。在这个过程中，我们获得了一份 CA 私钥文件 ./demoCA/private/cakey.pem 以及一份由此私钥签名的 CA 根证书文件 ./demoCA/cacert.pem ，得到的 CA 目录结构如下：
    ```
    demoCA/
    ├── cacert.pem
    ├── careq.pem
    ├── index.txt
    ├── index.txt.attr
    ├── index.txt.old
    ├── newcerts
    │   └── 01.pem
    ├── private
    │   └── cakey.pem
    ├── serial
    └── serial.old
    ```
5.  以上两个步骤可以合二为一
    ```
    $ openssl req -new -x509 -days 365 -key ./demoCA/private/cakey.pem -sha256 -extensions v3_ca -out ./demoCA/cacert.pem
    ```
    得到的目录结构如下：
    ```
    demoCA/
    ├── cacert.pem
    ├── index.txt
    ├── newcerts
    ├── private
    │   └── cakey.pem
    └── serial
    ```

6.  签发证书
    * 生成用户证书私钥  
      参照 CA 的私钥生成过程，使用如下命令生成新的私钥：
      ```
      $ openssl genrsa -aes256 -out userkey.pem
      ```
    * 生成用户证书请求  
      ```
      $ openssl req -new -days 365 -key userkey.pem -out userreq.pem
      Enter pass phrase for userkey.pem:
      You are about to be asked to enter information that will be incorporated into your certificate request.
      What you are about to enter is what is called a Distinguished Name or a DN.
      There are quite a few fields but you can leave some blank
      For some fields there will be a default value,
      If you enter '.', the field will be left blank.
      -----
      Country Name (2 letter code) [AU]:CN
      State or Province Name (full name) [Some-State]:Sichuan
      Locality Name (eg, city) []:Chengdu
      Organization Name (eg, company) [Internet Widgits Pty Ltd]:Phicomm
      Organizational Unit Name (eg, section) []:SmartHome
      Common Name (e.g. server FQDN or YOUR name) []:user
      Email Address []:step2hell@qq.com

      Please enter the following 'extra' attributes to be sent with your certificate request
      A challenge password []:
      An optional company name []:
      ```
    * 签发用户证书  
      现在，我们可以用先前建立的根 CA 来对用户的证书请求进行签名来为用户签发证书了。使用如下命令：
      ```
      $ openssl ca -in userreq.pem -out usercert.pem
      ```
      在`/usr/lib/ssl/openssl.cnf`中定义了 该命令使用默认路径生成的根证书 `./demoCA/cacert.pem` 以及根证书私钥`./demoCA/private/cakey.pem`, 如果根证书放在其他位置，可以使用参数 `-cert file`使用指定位置的根证书，同理用参数`-keyfile file`使用指定位置的根证书私钥。  

7.  至此，我们便完成了 CA 的建立及用户证书签发的全部工作。  
    下面，我们验证一下刚才用户证书的真实性：  
    ```
    $ openssl verify -CAfile ./demoCA/cacert.pem ./usercert.pem
    ./usercert.pem: OK
    ```
    结果显示 OK 。

8.  创建中间CA并通过中间CA颁发用户证书  
    创建中间CA的优点在我们不使用CA根私钥而是使用中间CA的私钥来签发客户证书，CA根私钥只用来签发中间CA的证书撤销列表，这样能够更为妥善地保护CA根私钥。  
    TODO

9.  用于[Raidus](https://tools.ietf.org/html/rfc2865) server端和client端相互认证的证书  

    仿照步骤6我们又用根证书生成两个用户证书： servercert.pem 和 clientcert.pem (当然也包含各自对应的私钥 serverkey.pem 和 clientkey.pem )。  

    由于 servercert.pem 和 clientcert.pem 都是由根证书签发的，处于同级关系，所以是没有办法相互认证的：
    ```
    $ openssl verify -CAfile ./servercert.pem ./clientcert.pem
    ./clientcert.pem: C = CN, ST = Sichuan, O = Phicomm, OU = SmartHome, CN = client, emailAddress = step2hell@qq.com
    error 20 at 0 depth lookup:unable to get local issuer certificate
    $ openssl verify -CAfile ./clientcert.pem ./servercert.pem
    ./servercert.pem: C = CN, ST = Sichuan, O = Phicomm, OU = SmartHome, CN = server, emailAddress = step2hell@qq.com
    error 20 at 0 depth lookup:unable to get local issuer certificate
    ```
    这个时候我们可以将两证书和各自的私钥以及根证书合并一起形成新的证书，两新证书便可相互认证了：  
    ```
    $ cat servercert.pem serverkey.pem ./demoCA/cacert.pem > server.pem
    $ cat clientcert.pem clientkey.pem ./demoCA/cacert.pem > client.pem
    $ openssl verify -CAfile server.pem client.pem
    client.pem: OK
    $ openssl verify -CAfile client.pem server.pem
    server.pem: OK
    ```
    将新证书 server.pem 和 client.pem 分别配置在 server端和client端即可。

    在测试安装证书的时候发现 Android手机目前不支持pem格式的CA证书，需要将其转化为crt格式[https://www.jethrocarr.com/2012/01/04/custom-ca-certificates-and-android/](https://www.jethrocarr.com/2012/01/04/custom-ca-certificates-and-android/)：
    ```
    $ openssl x509 -inform PEM -outform DER -in server.pem -out server.crt
    $ openssl x509 -inform PEM -outform DER -in client.pem -out client.crt
    ```
    但是转化为crt格式后的证书却无法相互认证：
    ```
    openssl verify -CAfile server.crt client.crt
    Error loading file server.crt
    ```
    目前还没找到更好的办法，只有重新生成一套crt格式的证书。


#   生成crt格式证书

1.  生成根证书私钥  
    ```
    $ sudo openssl genrsa -aes256 -out ca.key 4096
    ```
2.  生成根证书请求文件
    ```
    $ sudo openssl req -new -days 365 -key ca.key -sha256 -extensions v3_req -out ca.csr
    ```
3.  自签名根证书请求文件生成根证书
    ```
    $ sudo openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
    ```

    **Note**  
    以上三步可以用如下一行命令代替：
    ```
    $ sudo openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout ca.key -out ca.crt
    ```
    `-nodes`参数指定私钥不加密，如果需要私钥加密可以去掉该参数。  
    `req`指令具体参数可以参考[https://blog.csdn.net/as3luyuan123/article/details/16811787](https://blog.csdn.net/as3luyuan123/article/details/16811787)

4.  签发证书
    * 生成用户证书私钥
      ```
      $ sudo openssl genrsa -aes256 -out user.key 4096
      ```
    * 生成用户证书请求
      ```
      $ sudo openssl req -new -days 365 -key client.key -out user.csr
      ```
    * 签发用户证书
      ```
      $ sudo openssl x509 -req -days 365 -in user.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out user.crt
      ```
      该指令也可以用如下行代替（区别是下面的指令需要提前建好`/usr/lib/ssl/openssl.cnf`指定的一些生存文件夹及文件）：
      ```
      $ sudo openssl ca -in user.csr -out user.crt -cert ca.crt -keyfile ca.key
      ```
5.  证书验证
    ```
    $ sudo openssl verify -CAfile ca.crt user.crt
    user.crt: OK
    ```

    重复上述操作签发证书server.crt 和 client.crt，下面我们进行服务器端和客户端证书认证测试：
    * 单向认证测试
      在终端1 启动server：
      ```
      $ sudo openssl s_server -accept 10001 -key server.key -cert server.crt

      Using default temp DH parameters
      Using default temp ECDH parameters
      ACCEPT
      ```
      在终端2 启动客户端：
      ```
      $ sudo openssl s_client -connect localhost:10001

      CONNECTED(00000003)
      ```
      连接成功后在任意一个窗口输入字符串会传输到另外一个窗口回显。

    * 双向认证测试
      在终端1 启动server（带有Verify参数，强制要求client证书）：
      ```
      $ sudo openssl s_server -accept 10001 -key server.key -cert server.crt  -Verify 5
      ```
      在终端2 启动客户端：
      ```
      $ sudo openssl s_client -connect localhost:10001 -cert client.crt -key client.key
      ```
      双向证书正确则连接成功，否则连接失败。连接成功后同样在任意一个窗口输入字符串会传输到另外一个窗口回显。


参考：  
    [http://rhythm-zju.blog.163.com/blog/static/310042008015115718637/](http://rhythm-zju.blog.163.com/blog/static/310042008015115718637/)  
    [https://www.cnblogs.com/Security-Darren/p/4078867.html](https://www.cnblogs.com/Security-Darren/p/4078867.html)  
    [http://www.cnblogs.com/Security-Darren/p/4079605.html](http://www.cnblogs.com/Security-Darren/p/4079605.html)  
    [https://blog.csdn.net/lassewang/article/details/9159543](https://blog.csdn.net/lassewang/article/details/9159543)

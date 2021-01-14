创建自签名证书命令：
```
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout xxx.key -out xxx.crt
```

1. openssl：这是用于创建和管理OpenSSL证书、密钥和其他文件的基本命令工具。
2. req：此子命令指定我们要使用X.509证书签名请求管理。“X.509”是SSL和TLS为其密钥和证书管理所遵循的公钥基础结构标准。我们想要创建一个新的X.509证书，所以我们使用这个子命令。
3. -x509：这通过告诉实用程序我们要创建自签名证书而不是生成证书签名请求来进一步修改上一个子命令。
4. -nodes：这告诉OpenSSL跳过用密码保护我们的证书的选项。当服务器启动时，我们需要Nginx能够在没有用户干预的情况下读取文件。密码短语会阻止这种情况发生，因为我们必须在每次重启后输入密码。
5. -days 365：此选项设置证书的有效时间长度。我们在这里设置了十年。
6. -newkey rsa：2048：这指定我们要同时生成新证书和新密钥。我们没有创建在上一步中签署证书所需的密钥，因此我们需要将其与证书一起创建。该rsa:2048部分告诉它制作一个2048位长的RSA密钥。
7. -keyout：这一行告诉OpenSSL在哪里放置我们正在创建的生成的私钥文件。
8. -out：这告诉OpenSSL在哪里放置我们正在创建的证书。


这些选项将创建密钥文件和证书。工具将询问有关我们服务器的一些问题，以便将信息正确地填入到证书中。

最重要的一行是请求Common Name (e.g. server FQDN or YOUR name)。您需要输入与服务器关联的域名，或者是您服务器的公共IP地址。

整个提示将如下所示：
```
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.
Organizational Unit Name (eg, section) []:Ministry of Water Slides
Common Name (e.g. server FQDN or YOUR name) []:server_IP_address
Email Address []:admin@your_domain.com
```

输入相关信息后， 您将生成.key文件和.crt文件。
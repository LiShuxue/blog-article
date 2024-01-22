在安卓打包 release 的 .apk 文件 时，是需要选择一个 .keystore 的，即安卓数字签名证书。

```sh
keytool -genkey -keystore journey.keystore -alias journey -keyalg RSA -validity 36500 -storepass journey
```

- -keystore：指定生成的密钥库文件的名称。

- -storepass：设置密钥库的访问密码。

- -keypass：设置密钥（私钥）的密码。如果未指定，则默认与密钥库密码相同。

- -alias：设置别名，用于标识密钥对的唯一性。

- -keyalg：指定密钥的算法。常见的包括 RSA、DSA、EC。

- -sigalg：指定用于签名证书的算法。通常不需要指定，除非你有特定的需求。

- -keysize：指定密钥大小（位数）。例如，2048 或 4096。

- -validity：指定证书的有效期（以天为单位）。

剩下的全输入 journey 即可。

查看证书内容：

```sh
keytool -list -v -keystore journey.keystore

# 输入密码journey，回车
```

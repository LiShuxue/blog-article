## Nikto

Nikto 是一个开源的网络服务器扫描程序，它针对网络服务器的多个项目执行全面测试，包括 6700 多个潜在危险文件/程序，检查 1250 多个服务器的过时版本，以及 270 多个服务器上的版本特定问题。它还会检查服务器配置项，例如是否存在多个索引文件、HTTP 服务器选项，并将尝试识别已安装的 Web 服务器和软件。

## 使用方式

进入 git 页面：<https://github.com/sullo/nikto，参考步骤>

我使用了 nikto-2.1.6，当时 2.5.0 不能成功。

执行：`perl nikto.pl -h https://lishuxue.site -o result.html -F html` 会开始扫描，并将结果输出到 result.html 中。

![scanstep](https://cdn.lishuxue.site/blog/image/ECS/scanstep.png)

## 结果

根据扫描结果，一个个处理问题。

![result](https://cdn.lishuxue.site/blog/image/ECS/result.png)

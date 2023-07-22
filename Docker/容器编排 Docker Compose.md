## Docker Compose 简介

使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的服务容器，数据库服务容器，甚至还包括负载均衡容器等。如果只使用 Dockerfile，我们需要操作每一个容器的启动停止。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。一次性的管理这些容器。

Compose 中有两个重要的概念：

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

## 命令

在有 docker-compose.yml 的目录，执行下面命令。

```sh
# docker compose up -d 启动
# docker compose down 停止
# docker compose ps 查看启动的容器，也可以用 docker ps 查看
# docker compose up -d service_name 单独启动一个容器
# docker compose stop service_name
# docker compose build 只构建镜像，可以加service_name来单独构建某个服务的镜像
```

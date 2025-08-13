# Nacos + Higress 的 MCP 开发新范式

Naocs 3.0 中已经支持和Higress配合使用实现存量Http转化为MCP服务，3.0.1 及以上版本在支持存量转化的基础上同时支持同步Nacos中已经注册的原生的MCP服务，并在Higress上进行暴露，完成了对所有Remote Server类型的代理访问支持。

![img](https://img.alicdn.com/imgextra/i4/O1CN01cIKoQq1OJK3C0Z7wW_!!6000000001684-2-tps-1113-369.png)

通过结合 Spring AI Alibaba，FastMCP 等框架，可以实现应用自动注册到Nacos中，并通过 Higress 自动将注册的应用对外暴露给Client侧访问。此文档从0到一完成Higress+Nacos配合实现REST API转MCP和透明代理暴露标准 MCP 服务。

## 环境准备

### 创建独立的docker网络

```
docker network create mcp
```

### Higress 部署

使用docker部署Higress

```
docker run -d --rm --name higress-ai -v ${PWD}:/data \
        -p 8001:8001 -p 8080:8080 -p 8443:8443 --network mcp \
        higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/all-in-one:latest
```

部署redis

```
docker run -d --rm --name higress-redis -p 6379:6379 --network mcp higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/redis-stack-server:7.4.0-v3
```

配置MCP Server的全局参数

```
vi ./configmaps/higress-config.yaml
```

修改配置文件

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: higress-config
  namespace: higress-system
  creationTimestamp: "2000-01-01T00:00:00Z"
  resourceVersion: "1"
data:
  higress: |-
    mcpServer:
      sse_path_suffix: /sse  # SSE 连接的路径后缀
      enable: true          # 启用 MCP Server
      redis:
        address: IP:6379 # Redis服务地址。这里需要使用本机的内网 IP，不可以使用 127.0.0.1
        username: "" # Redis用户名（可选）
        password: "" # Redis密码（可选）
        db: 0 # Redis数据库（可选）
      match_list:          # MCP Server 会话保持路由规则（当匹配下面路径时，将被识别为一个 MCP 会话，通过 SSE 等机制进行会话保持,可选）
        - match_rule_domain: "*"
          match_rule_path: /user
          match_rule_type: "prefix"
      servers: []
    downstream:
    # 以下配置无需修改，此处省略
```

> [!NOTE]
>
> 受 Docker 运行环境的限制，非 Linux 操作系统在修改 yaml 文件之后，需要等待一段时间才能让新的配置生效。如果希望立即生效的话，可以使用以下命令重启 higress-ai 容器：
>
> ```
> docker stop higress-ai
> docker run -d --rm --name higress-ai -v ${PWD}:/data -e O11Y=on \
>         -p 8001:8001 -p 8080:8080 -p 8443:8443 \
>         higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/all-in-one:latest
> ```
>
> 

### Nacos部署

通过docker直接部署

```
docker run --name nacos \
    -e MODE=standalone \
    -e NACOS_AUTH_TOKEN=your_token_base64 \
    -e NACOS_AUTH_IDENTITY_KEY=your_key \
    -e NACOS_AUTH_IDENTITY_VALUE=your_value \
    -p 8081:8080 \
    -p 8848:8848 \
    -p 9848:9848 \
    --network mcp \
    -d nacos-registry.cn-hangzhou.cr.aliyuncs.com/nacos/nacos-server:v3.0.1
```

> [!NOTE]
>
> example
>
> ```
> docker run --name nacos \
>     -e MODE=standalone \
>     -e NACOS_AUTH_TOKEN=U7bzqnaqqov0s/VsuBclzXKInNHjsmQ8c4b1bYZQ//0= \
>     -e NACOS_AUTH_IDENTITY_KEY=123456 \
>     -e NACOS_AUTH_IDENTITY_VALUE=123456 \
>     -p 8081:8080 \
>     -p 8848:8848 \
>     -p 9848:9848 \
>     --network mcp \
>     -d nacos-registry.cn-hangzhou.cr.aliyuncs.com/nacos/nacos-server:v3.0.1
> ```
>
> 



### 配置Higress 连接到Nacos mcp registry

在Higress 服务来源中增加Nacos3.x服务来源

打开 higress 控制台，部署主机地址:8001, 点击服务来源添加，并添加nacos3.x 服务来源，地址填写 nacos

> [!CAUTION]
>
> 需要Nacos版本为3.0及以上，Higress版本在2.1.2及以上

新增服务来源

![添加服务来源](https://img.alicdn.com/imgextra/i3/O1CN01Ksd48C1ru4g6ep9SU_!!6000000005690-2-tps-2422-198.png)

创建nacos3.x服务来源并完善相关信息

![添加Nacos3.x服务来源](https://img.alicdn.com/imgextra/i3/O1CN01FGvSE71HVVGHTp1Cu_!!6000000000763-2-tps-588-1039.png)

通过以下规则访问接入点

- REST API 转 MCP 类型。http://{关联MCP Server的域名}/{MCP Server路由路径前缀}/{Nacos中MCP Server的名称}，例如关联Nacos服务来源关联的域名是mcp-registry.com, mcp server 路由路径 为 /mcp，Nacos中mcp服务的名称为test，则最终的sse协议的访问路径为 http://mcp-registry.com/mcp/test/sse
- 直接代理原生 MCP 类型。http://{关联MCP Server的域名}/{MCP Server路由路径前缀}/{Nacos中MCP Server的名称}/{原生服务的sse路径}，例如关联Nacos服务来源关联的域名是mcp-registry.com, mcp server 路由路径 为 /mcp，Nacos中mcp服务的名称为test，test服务原本的sse路径为/mcp/sse，则最终的sse协议访问路径为 http://mcp-registry.com/mcp/test/mcp/sse

### 配置 REST API MCP Server

任何 REST API 都可以通过以下步骤快速转换为 MCP Server：

#### 1. 打开 Higress 控制台

在浏览器中访问 `http://localhost:8001`。

首次访问将会要求配置登录账号信息。配置完成后，请使用配置好的账号登录。

#### 2. 添加服务来源

在 Higress 控制台添加目标 REST API 的服务来源。本示例使用 `randomuser.me` 作为服务来源。

![添加服务来源-信息](https://higress.cn/img/ai/mcp-quick-start_docker/zh/add-servicesource-info.png)

添加完成后，服务来源列表显示如下：

![添加服务来源-列表](https://higress.cn/img/ai/mcp-quick-start_docker/zh/add-servicesource-list.png)

#### 3. 配置路由

在 Higress 控制台添加路由并指向对应的服务来源：

![配置路由-信息](https://higress.cn/img/ai/mcp-quick-start_docker/zh/add-route-info.png)

添加完成后，路由列表显示如下：

![配置路由-列表](https://higress.cn/img/ai/mcp-quick-start_docker/zh/add-route-list.png)

#### 3. 配置 MCP Server 插件

1. 点击新创建的路由右侧的“策略”链接进入插件配置页面。
2. 找到“MCP 服务器”插件，并点击其下方的配置按钮
3. 将“开启状态”切换至绿色的开。
4. 将下发配置部分切换到 YAML 视图，并填入以下配置。
5. 点击右上角的保存。

```yaml
server:
  name: "random-user-server"
tools:
- description: "Get random user information"
  name: "get-user"
  requestTemplate:
    method: "GET"
    url: "https://randomuser.me/api/"
  responseTemplate:
    body: |-
      # User Information
      {{- with (index .results 0) }}
      - **Name**: {{.name.first}} {{.name.last}}
      - **Email**: {{.email}}
      - **Location**: {{.location.city}}, {{.location.country}}
      - **Phone**: {{.phone}}
      {{- end }}
```

![配置MCP Server插件](https://gw.alicdn.com/imgextra/i4/O1CN01HLJt6I26ehbSLSA57_!!6000000007687-0-tps-2940-1184.jpg)

## MCP Server 使用

在 AI Agent 中配置 MCP Server 的 SSE Server，本文以 [Cherry Studio](https://cherry-ai.com/) 为例。

### 1. 添加 MCP 服务器

按照以下说明添加一个新的 MCP 服务器指向我们刚刚配置的路由：

1. 名称：可以任意填写
2. 类型：`服务器发送事件（sse）`
3. URL：`http://localhost:8080/user/sse`

![配置MCP Server插件-1](https://higress.cn/img/ai/mcp-quick-start_docker/zh/cherry-studio-add-mcp-1.png)

![配置MCP Server插件-2](https://higress.cn/img/ai/mcp-quick-start_docker/zh/cherry-studio-add-mcp-2.png)

### 2. 使用 MCP 服务

在 Cherry Studio 中，要在对话中使用 MCP 服务，我们需要先激活它。

1. 切换到“助手”页面
2. 点击右侧输入框下方的“MCP 服务器”按钮
3. 在弹出的 MCP 服务器列表中点击我们刚刚添加的“GetRandomUser”服务，使其变绿且右侧出现“√”标识

![激活MCP Server](https://higress.cn/img/ai/mcp-quick-start_docker/zh/activate-mcp-server.png)

现在我们就可以在对话中使用这个 MCP 服务服务了。

在输入框中输入“帮我生成一段用户信息”。就可以看到 Cherry Studio 利用大模型并通过 MCP Server 来为我们生成了一段随机的用户信息。

![使用MCP Server](https://higress.cn/img/ai/mcp-quick-start_docker/zh/use-mcp-server.png)

## 常见问题

### 1. higress-config 中的 mcpServer 配置与路由配置以及路由策略中的 MCP 服务器插件之间是什么关系？

`higress-config` 中的 `mcpServer` 配置是用来进行 SSE 数据流推送的。所以它里面的 `matchList` 中的匹配规则必须要覆盖所有使用 SSE 协议的 MCP 服务器路径。

路由配置是用来让请求能够正常被转发到目标 REST API 的。所以路由所指向的目标服务必须是 MCP 服务所对应的 REST API。

路由策略配置中的 MCP 服务器插件配置有两个功能，一个是向 MCP 客户端（也就是例子中的 Cherry Studio）提供 MCP 服务所支持的工具方法的元数据，另一个则是在转发请求的时候将 MCP 请求转换成目标服务可以支持的格式，也就是 `requestTemplate` 部分所配置的内容。

### 2. 是否可以不用部署 Redis？

由于 MCP 协议发起工具调用和推送响应所对应的是不同的请求，甚至在集群部署的时候，这些请求可能会由集群中的不同服务器进行处理，所以 Higress 引入了 Redis 来进行会话管理，确保所有的工具调用都可以关联到最初开启会话的 SSE 请求。

如果不想用 Redis 的话，可以尝试一下 MCP 协议中新推出的 Streamble HTTP 通信方式。

### 3. 如何查看日志？

Higress all-in-one 容器的所有日志都保存在容器中的 `/var/log/higress` 目录中。大家可以使用以下命令进入到容器中进行查看。

```bash
docker exec -it higress-ai bash
cd /var/log/higress
cat gateway.log
```

## 总结

通过 Higress 的 MCP Server 功能，您可以快速为 AI Agent 添加各种数据源支持，提高工作效率。任何 REST API 都可以通过简单的配置转换为 MCP Server，无需编写额外的代码。
# Dify-V1.7.2在windows上基于wsl2进行源码部署

##  前提条件

### 设置 Docker 和 Docker Compose

> 在安装 Dify 之前，请确保您的设备符合以下最低系统要求：
>
> - CPU >= 2 核
> - RAM >= 4 GiB

1. 首先需要安装wsl2
2. 安装Docker Desktop
3. 在Docker Desktop中配置使得wsl2中可以运行docker命令
   - Docker 也专门开发了可以使用 `WSL2` 中的 `Docker` 守护进程的桌面管理程序, 打开 Docker Desktop WSL2 backend 页面，下载最新的 [Docker Desktop for Windows](https://zhida.zhihu.com/search?content_id=121135201&content_type=Article&match_order=1&q=Docker+Desktop+for+Windows&zhida_source=entity) 程序 ，建议下载stable版本。下载地址：[https://www.docker.com/products/docker-desktop](https://link.zhihu.com/?target=https%3A//www.docker.com/products/docker-desktop)
   - 启动Docker Desktop for Windows，点击“设置”按钮，启用基于`WSL2`的引擎复选框（`Use the WSL 2 based engine）`

​	![img](https://pica.zhimg.com/v2-dbd56c4406cc48801ca23880d1a95410_1440w.jpg)

这时候在 WSL 里面执行 docker 命令还是找不到的

在 Resources 的WSL Integration中设置要从哪个 WSL2 发行版中访问 Docker，如下图使用的是 Ubuntu。

![img](https://pic4.zhimg.com/v2-3bd962fcd1038ac10a3d15609646e631_1440w.jpg)

重启 Docker desktop for Windows，重启完成后我们就可以在 WSL2里面使用 docker 命令了



| 操作系统               | 软件                                                     | 说明                                                         |
| ---------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| macOS 10.14 或更高版本 | Docker Desktop                                           | 将 Docker 虚拟机(VM)设置为至少使用 2 个虚拟 CPU(vCPU)和 8 GB 的初始内存。否则，安装可能会失败。更多信息，请参考 [Docker Desktop for Mac 安装指南](https://docs.docker.com/desktop/mac/install/)。 |
| Linux 平台             | Docker 19.03 或更高版本 Docker Compose 1.25.1 或更高版本 | 请参考 [Docker 安装指南](https://docs.docker.com/engine/install/) 和 [Docker Compose 安装指南](https://docs.docker.com/compose/install/) 了解如何分别安装 Docker 和 Docker Compose。 |
| 启用 WSL 2 的 Windows  | Docker Desktop                                           | 我们建议将源代码和绑定到 Linux 容器的其他数据存储在 Linux 文件系统中，而不是 Windows 文件系统中。更多信息，请参考 [Windows 上使用 WSL 2 后端的 Docker Desktop 安装指南](https://docs.docker.com/desktop/windows/install/#wsl-2-backend)。 |

> 如果需要使用 OpenAI TTS，系统必须安装 `FFmpeg` 才能正常运行。更多详情，请参考：[链接](https://docs.dify.ai/getting-started/install-self-hosted/install-faq#id-14.-what-to-do-if-this-error-occurs-in-text-to-speech)。

### 克隆 Dify 仓库

运行 git 命令克隆 [Dify 仓库](https://github.com/langgenius/dify)。

```
git clone https://github.com/langgenius/dify.git
```

### 使用 Docker Compose 启动中间件

Dify 后端服务需要一系列用于存储（如 PostgreSQL / Redis / Weaviate（如果本地不可用））和扩展能力（如 Dify 的 [sandbox](https://github.com/langgenius/dify-sandbox) 和 [plugin-daemon](https://github.com/langgenius/dify-plugin-daemon) 服务）的中间件。通过运行以下命令使用 Docker Compose 启动中间件：

```
cd docker
cp middleware.env.example middleware.env
docker compose -f docker-compose.middleware.yaml up -d
```

### 一些启动依赖的中间件异常问题

#### 1.nvidia依赖缺失相关问题

```
error1.unknown or invalid runtime name: nvidia 
error2.error during container init: error running hook #1: fork/exec /usr/bin/nvidia-container-runtime-hook: no such file or directory: unknown
```

![img](https://upload-images.jianshu.io/upload_images/1760913-4e46b1f7975b6aac.png?imageMogr2/auto-orient/strip|imageView2/2/w/1034/format/webp)

![img](https://upload-images.jianshu.io/upload_images/1760913-31dae429dee3c287.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

参考如何在Ubuntu 22.04离线安装Docker和NVIDIA Container Toolkit

#### 2.docker-db-1无法成功启动

```
initdb: error: could not change permissions of directory"/var/lib/postgresql/data/pgdata"

```

在windows上用docker desktop + wsl2（我用的ubuntu）部署dify时，如果把dify的项目文件放在d盘（在ubuntu下的目录是/mnt/d），此时直接docker compose up -d启动容器， 会导致 postgre数据库不断重启，并报如下的权限问题：

```kotlin
db-1             | chmod: /var/lib/postgresql/data/pgdata: Operation not permitted 
db-1             | The files belonging to this database system will be owned by user "postgres". 
db-1             | This user must also own the server process.
```

具体原因跟据 [官方问答中的评论](https://github.com/langgenius/dify/issues/4621)，似乎是docker-compose.yml文件中，表明了容器内postgre数据库的相关文件存储地址 /var/lib/postgresql/data/pgdata 映射到了宿主机的 D:/dify-main/volumes/db/data上， 而又因为D盘本质上属于window系统，wsl2没有相应的权限去进行操作，所以导致刚才所述的问题, 解决方法就是改变映射，让他直接映射到wsl2的存储空间

另一个解释是来自[这篇评论](https://github.com/docker-library/postgres/issues/558)

从运行 Docker for Windows 的 VM 的角度来看，任何来自宿主机的共享目录都由 root（ [docker/for-win#63](https://github.com/docker/for-win/issues/63) ）拥有。如果我记得没错的话，任何用户都可以读写这些目录。不幸的是，Postgres 必须以目录的所有者身份运行（无论读写是否成功），而不能以 root 身份运行。因此，唯一的解决方案是使用命名卷而不是将共享目录提供给 Windows 宿主机。

##### 解决方案1：

- 将容器内postgre 数据库存储地址，映射到wsl2中的存储空间，例如/home 文件夹

- 具体操作如下

  - 打开docker-compose.yml文件

  - 找到postgre数据库对应的一段

    ```yaml
    # The postgres database.
      db:
        image: postgres:15-alpine
        restart: always
        environment:
          PGUSER: ${PGUSER:-postgres}
          POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-difyai123456}
          POSTGRES_DB: ${POSTGRES_DB:-dify}
          PGDATA: ${PGDATA:-/var/lib/postgresql/data/pgdata}
        command: >
          postgres -c 'max_connections=${POSTGRES_MAX_CONNECTIONS:-100}'
                   -c 'shared_buffers=${POSTGRES_SHARED_BUFFERS:-128MB}'
                   -c 'work_mem=${POSTGRES_WORK_MEM:-4MB}'
                   -c 'maintenance_work_mem=${POSTGRES_MAINTENANCE_WORK_MEM:-64MB}'
                   -c 'effective_cache_size=${POSTGRES_EFFECTIVE_CACHE_SIZE:-4096MB}'
        volumes:
          - ./volumes/db/data:/var/lib/postgresql/data
        healthcheck:
          test: [ 'CMD', 'pg_isready' ]
          interval: 1s
          timeout: 3s
          retries: 30
        ports:
          - '${EXPOSE_DB_PORT:-5432}:5432'
    ```

  - 修改映射关系

    ```ruby
    #找到volumes字段
    volumes:
          - ./volumes/db/data:/var/lib/postgresql/data
    #改为
    volumes:
          - /home/<你自己的文件夹>:/var/lib/postgresql/data
        
    ```

##### 解决方案2：[来自这篇文章的评论](https://github.com/docker-library/postgres/issues/558)

Ok, so here is a solution that helped me. First, I ran the command `docker volume create --name=pgdata` in the terminal. Then my co-worker (who has a lot more experience with Docker than me, who has barely any) helped me modify my docker-compose.yml file so that it included the following:

```yaml
  services:
      db:
          image: postgres
          restart: unless-stopped
          env_file:
            - .env
          volumes:
            - pgdata:/var/lib/postgresql/data
          ports:
            - "5434:5432"
  volumes:
    pgdata:
      external: true
```

Note: the volumes block is indented at the same level as services. This apparently helps tell Docker that the particular folder being used for data storage is defined as a volume, which has a different set of access rights versus a folder. I'm not entirely sure if this is the correct way to explain it, but my error of `data directory "/var/lib/postgresql/data" has invalid permission` has ceased and the app is now working.

##### 解决方案3：

The reason is you download the source code in your **Windows** system, but run docker in **wsl2**.
The correct opreation is:

- git clone in wsl2, not in windows
- docker pull
- dock compose

原因是docker使用wsl2后端，【wsl环境的docker环境里面的程序】无权限访问【windows环境下载的dify文件夹】，重新在wsl下git就能拉起。

------

## 设置后端服务

后端服务包括：

1. API 服务：为前端服务和 API 访问提供 API 请求服务
2. Worker 服务：为数据集处理、工作区、清理等异步任务提供服务

### 环境准备

需要 Python 3.12。推荐使用 [pyenv](https://github.com/pyenv/pyenv) 快速安装 Python 环境。要安装额外的 Python 版本，请使用 pyenv install。

### 启动 API 服务

1. 导航到 `api` 目录：

   ```
   cd api
   ```

2. 准备环境变量配置文件

   ```
   cp .env.example .env
   ```

3. 生成随机密钥并替换 .env 文件中的 SECRET_KEY 值

   ```
   awk -v key="$(openssl rand -base64 42)" '/^SECRET_KEY=/ {sub(/=.*/, "=" key)} 1' .env > temp_env && mv temp_env .env
   ```

4. 安装依赖

   uv安装教程参考uv官网:[安装 | uv 中文文档](https://uv.doczh.com/getting-started/installation/)

   使用 [uv](https://docs.astral.sh/uv/getting-started/installation/) 管理依赖。 通过运行以下命令使用 `uv` 安装所需依赖：

   ```
   uv sync
   ```

   > 对于 macOS：使用 `brew install libmagic` 安装 libmagic。

5. 执行数据库迁移

   执行数据库迁移到最新版本：

   ```
   uv run flask db upgrade
   ```

6. 启动 API 服务

   ```
   uv run flask run --host 0.0.0.0 --port=5001 --debug
   ```

   预期输出：
   ```
   * Debug mode: on
   INFO:werkzeug:WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
    * Running on all addresses (0.0.0.0)
    * Running on http://127.0.0.1:5001
   INFO:werkzeug:Press CTRL+C to quit
   INFO:werkzeug: * Restarting with stat
   WARNING:werkzeug: * Debugger is active!
   INFO:werkzeug: * Debugger PIN: 695-801-919
   ```



### 启动 Worker 服务

要从队列中消费异步任务，例如数据集文件导入和数据集文档更新，请按照以下步骤启动 Worker 服务：

- 对于 macOS 或 Linux

  ```
  uv run celery -A app.celery worker -P gevent -c 1 --loglevel INFO -Q dataset,generation,mail,ops_trace
  ```

  如果您使用 Windows 系统启动 Worker 服务，请使用以下命令：

- 对于 Windows

  ```
  uv run celery -A app.celery worker -P solo --without-gossip --without-mingle -Q dataset,generation,mail,ops_trace --loglevel INFO
  ```

  预期输出：

  ```
  -------------- celery@bwdeMacBook-Pro-2.local v5.4.0 (opalescent)
  --- ***** -----
  -- ******* ---- macOS-15.4.1-arm64-arm-64bit 2025-04-28 17:07:14
  - *** --- * ---
  - ** ---------- [config]
  - ** ---------- .> app:         app_factory:0x1439e8590
  - ** ---------- .> transport:   redis://:**@localhost:6379/1
  - ** ---------- .> results:     postgresql://postgres:**@localhost:5432/dify
  - *** --- * --- .> concurrency: 1 (gevent)
    -- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
    --- ***** -----
    -------------- [queues]
    .> dataset          exchange=dataset(direct) key=dataset
    .> generation       exchange=generation(direct) key=generation
    .> mail             exchange=mail(direct) key=mail
    .> ops_trace        exchange=ops_trace(direct) key=ops_trace
  
  [tasks]
  . schedule.clean_embedding_cache_task.clean_embedding_cache_task
  . schedule.clean_messages.clean_messages
  . schedule.clean_unused_datasets_task.clean_unused_datasets_task
  . schedule.create_tidb_serverless_task.create_tidb_serverless_task
  . schedule.mail_clean_document_notify_task.mail_clean_document_notify_task
  . schedule.update_tidb_serverless_status_task.update_tidb_serverless_status_task
  . tasks.add_document_to_index_task.add_document_to_index_task
  . tasks.annotation.add_annotation_to_index_task.add_annotation_to_index_task
  . tasks.annotation.batch_import_annotations_task.batch_import_annotations_task
  . tasks.annotation.delete_annotation_index_task.delete_annotation_index_task
  . tasks.annotation.disable_annotation_reply_task.disable_annotation_reply_task
  . tasks.annotation.enable_annotation_reply_task.enable_annotation_reply_task
  . tasks.annotation.update_annotation_to_index_task.update_annotation_to_index_task
  . tasks.batch_clean_document_task.batch_clean_document_task
  . tasks.batch_create_segment_to_index_task.batch_create_segment_to_index_task
  . tasks.clean_dataset_task.clean_dataset_task
  . tasks.clean_document_task.clean_document_task
  . tasks.clean_notion_document_task.clean_notion_document_task
  . tasks.deal_dataset_vector_index_task.deal_dataset_vector_index_task
  . tasks.delete_account_task.delete_account_task
  . tasks.delete_segment_from_index_task.delete_segment_from_index_task
  . tasks.disable_segment_from_index_task.disable_segment_from_index_task
  . tasks.disable_segments_from_index_task.disable_segments_from_index_task
  . tasks.document_indexing_sync_task.document_indexing_sync_task
  . tasks.document_indexing_task.document_indexing_task
  . tasks.document_indexing_update_task.document_indexing_update_task
  . tasks.duplicate_document_indexing_task.duplicate_document_indexing_task
  . tasks.enable_segments_to_index_task.enable_segments_to_index_task
  . tasks.mail_account_deletion_task.send_account_deletion_verification_code
  . tasks.mail_account_deletion_task.send_deletion_success_task
  . tasks.mail_email_code_login.send_email_code_login_mail_task
  . tasks.mail_invite_member_task.send_invite_member_mail_task
  . tasks.mail_reset_password_task.send_reset_password_mail_task
  . tasks.ops_trace_task.process_trace_tasks
  . tasks.recover_document_indexing_task.recover_document_indexing_task
  . tasks.remove_app_and_related_data_task.remove_app_and_related_data_task
  . tasks.remove_document_from_index_task.remove_document_from_index_task
  . tasks.retry_document_indexing_task.retry_document_indexing_task
  . tasks.sync_website_document_indexing_task.sync_website_document_indexing_task
  
  2025-04-28 17:07:14,681 INFO [connection.py:22]  Connected to redis://:**@localhost:6379/1
  2025-04-28 17:07:14,684 INFO [mingle.py:40]  mingle: searching for neighbors
  2025-04-28 17:07:15,704 INFO [mingle.py:49]  mingle: all alone
  2025-04-28 17:07:15,733 INFO [worker.py:175]  celery@bwdeMacBook-Pro-2.local ready.
  2025-04-28 17:07:15,742 INFO [pidbox.py:111]  pidbox: Connected to redis://:**@localhost:6379/1.
  ```

------

## 设置 Web 服务

启动用于前端页面的 web 服务。

### 环境准备

要启动 web 前端服务，需要 [Node.js v22 (LTS)](http://nodejs.org/) 和 [PNPM v10](https://pnpm.io/)。

- 安装 NodeJS 请访问 https://nodejs.org/en/download 并选择适合您操作系统的 v18.x 或更高版本的安装包。推荐使用 LTS 版本进行常规使用。

- 安装 PNPM

  按照 [安装指南](https://pnpm.io/installation) 安装 PNPM。或者直接使用 `npm` 运行以下命令安装 `pnpm`。

  ```
  npm i -g pnpm
  ```

### 启动 Web 服务

1. 进入 web 目录

   ```
   cd web
   ```

2. 安装依赖

   ```
   pnpm install --frozen-lockfile
   ```

3. 准备环境变量配置文件

   在当前目录中创建一个名为 `.env.local` 的文件，并从 `.env.example` 复制内容。根据您的需求修改这些环境变量的值：

   ```
   # For production release, change this to PRODUCTION
   NEXT_PUBLIC_DEPLOY_ENV=DEVELOPMENT
   # The deployment edition, SELF_HOSTED or CLOUD
   NEXT_PUBLIC_EDITION=SELF_HOSTED
   # The base URL of console application, refers to the Console base URL of WEB service if console domain is
   # different from api or web app domain.
   # example: http://cloud.dify.ai/console/api
   NEXT_PUBLIC_API_PREFIX=http://localhost:5001/console/api
   # The URL for Web APP, refers to the Web App base URL of WEB service if web app domain is different from
   # console or api domain.
   # example: http://udify.app/api
   NEXT_PUBLIC_PUBLIC_API_PREFIX=http://localhost:5001/api
   
   # SENTRY
   NEXT_PUBLIC_SENTRY_DSN=
   NEXT_PUBLIC_SENTRY_ORG=
   NEXT_PUBLIC_SENTRY_PROJECT=
   ```

4. 构建 web 服务

   ```
   pnpm build
   ```

5. 启动 web 服务

   ```
   pnpm start
   ```

   预期输出：

   ```
      ▲ Next.js 15
      - Local:        http://localhost:3000
      - Network:      http://0.0.0.0:3000
   
    ✓ Starting...
    ✓ Ready in 73ms
   ```

### 访问 Dify

通过浏览器访问 [http://127.0.0.1:3000](http://127.0.0.1:3000/) 即可享受 Dify 所有激动人心的功能。干杯！🍻

# Docker 容器网络、数据卷、MySQL 与 Compose

## 一、整体结构

```text
浏览器
  ↓ 宿主机端口映射
Python / Flask 容器
  ↓ Docker 网络
MySQL 容器
  ↓ Volume 挂载
持久化数据库文件
```

第三课重点：容器网络、环境变量、MySQL 和 Volume。

第四课重点：使用 Docker Compose 统一管理多个容器。

---

## 二、Docker 容器网络

### 创建和查看网络

```powershell
docker network create app-network
docker network ls
docker network inspect app-network
```

`docker network inspect app-network` 的 `Containers` 部分会显示已加入网络的容器及其内部 IP。

将正在运行的容器接入网络：

```powershell
docker network connect app-network 容器名
```

断开网络：

```powershell
docker network disconnect app-network 容器名
```

### 容器如何互相访问

第三课中的两个容器：

```text
app-network
├── python-mysql-web
└── mysql-db
```

Python 通过下面的地址访问 MySQL：

```text
mysql-db:3306
```

- `mysql-db`：MySQL 容器名称。
- `3306`：MySQL 在容器内监听的默认端口。
- Docker 网络会把容器名称解析成对应的内部 IP。

### 为什么不能使用 `localhost`

在 Python 容器中：

```text
localhost = Python 容器自己
localhost ≠ MySQL 容器
```

因此，Python 需要使用 `mysql-db` 访问数据库。

### 为什么 MySQL 不需要映射端口

同一 Docker 网络中的容器可以直接使用容器名称和内部端口通信：

```text
python-mysql-web → mysql-db:3306
```

`-p` 主要用于宿主机访问容器：

```text
宿主机访问容器       → 需要端口映射
同一网络的容器互访   → 使用容器名和内部端口
```

如果需要用 Windows 上的数据库工具连接 MySQL，可以添加：

```powershell
-p 3307:3306
```

然后连接：

```text
主机：localhost
端口：3307
```

---

## 三、MySQL 容器与环境变量

### 启动 MySQL

```powershell
docker run -d --name mysql-db --network app-network -e MYSQL_ROOT_PASSWORD=root-demo-password -e MYSQL_DATABASE=docker_demo -e MYSQL_USER=appuser -e MYSQL_PASSWORD=app-demo-password -v mysql-data:/var/lib/mysql mysql:8.4
```

关键参数：

| 参数 | 作用 |
|---|---|
| `--name mysql-db` | 设置容器名称 |
| `--network app-network` | 加入 Docker 网络 |
| `MYSQL_ROOT_PASSWORD` | 设置 root 密码 |
| `MYSQL_DATABASE` | 首次启动时创建数据库 |
| `MYSQL_USER` | 创建普通数据库用户 |
| `MYSQL_PASSWORD` | 设置普通用户密码 |
| `-v mysql-data:/var/lib/mysql` | 持久化数据库文件 |

查看启动日志：

```powershell
docker logs -f mysql-db
```

看到 `ready for connections` 表示数据库可以接受连接。

### 登录 MySQL

```powershell
docker exec -it mysql-db mysql -uappuser -p
```

命令结构：

```text
docker exec -it  mysql-db  mysql  -uappuser  -p
                 │         │      │           │
                 │         │      │           └─ 提示输入密码
                 │         │      └─ 使用 appuser 登录
                 │         └─ 容器内执行的 MySQL 客户端程序
                 └─ 目标容器名称
```

`mysql-db` 是容器名，中间的 `mysql` 是容器内安装的命令行程序。

查看数据库：

```sql
SHOW DATABASES;
```

退出：

```sql
exit;
```

---

## 四、Volume 数据持久化

挂载格式：

```text
Volume名称:容器内部路径
```

MySQL 示例：

```text
mysql-data:/var/lib/mysql
```

- `mysql-data`：Docker 管理的命名 Volume。
- `/var/lib/mysql`：MySQL 容器内部保存数据库文件的默认目录。

查看 Volume：

```powershell
docker volume ls
docker volume inspect mysql-data
```

在 Windows Docker Desktop 中，Volume 通常保存在 Docker 管理的 Linux 虚拟环境中，不是普通的 Windows 项目目录，一般不应手动修改。

### 删除容器与删除数据的区别

```powershell
docker rm -f mysql-db
```

只删除容器，不会删除 `mysql-data`。

重新创建 MySQL 容器并挂载同一个 Volume 后，可以继续使用原数据：

```text
旧 MySQL 容器
      ↓
mysql-data Volume
      ↑
新 MySQL 容器
```

主动删除 Volume：

```powershell
docker volume rm mysql-data
```

删除 Volume 会删除其中保存的数据库数据。

---

## 五、Python 连接 MySQL

### Python 依赖

`requirements.txt`：

```text
Flask>=3.0,<4.0
PyMySQL>=1.1,<2.0
```

### 数据库连接

```python
def get_connection():
    return pymysql.connect(
        host=os.environ.get("DB_HOST", "mysql-db"),
        port=3306,
        user=os.environ.get("DB_USER", "appuser"),
        password=os.environ.get("DB_PASSWORD"),
        database=os.environ.get("DB_NAME", "docker_demo"),
        charset="utf8mb4",
    )
```

数据库配置通过环境变量传入：

```powershell
-e DB_HOST=mysql-db
-e DB_USER=appuser
-e DB_PASSWORD=app-demo-password
-e DB_NAME=docker_demo
```

这样可以在不同环境中使用不同配置，也能避免把密码直接打包进镜像。

> 环境变量仍可能被拥有 Docker 管理权限的人查看。生产环境通常使用专门的密钥管理工具。

### `home()` 的执行流程

```text
浏览器访问 /
    ↓
建立数据库连接
    ↓
创建 visits 表（表不存在时）
    ↓
插入一条访问记录
    ↓
commit 提交事务
    ↓
查询记录总数
    ↓
返回 HTML
    ↓
关闭数据库连接
```

关键代码：

```python
connection = get_connection()

try:
    with connection.cursor() as cursor:
        cursor.execute(
            """
            CREATE TABLE IF NOT EXISTS visits (
                id INT AUTO_INCREMENT PRIMARY KEY,
                visited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
            """
        )

        cursor.execute("INSERT INTO visits () VALUES ()")
        connection.commit()

        cursor.execute("SELECT COUNT(*) FROM visits")
        visit_count = cursor.fetchone()[0]
finally:
    connection.close()
```

重点：

- `connection`：Python 与数据库之间的连接。
- `cursor`：通过连接执行 SQL。
- `CREATE TABLE IF NOT EXISTS`：表不存在时才创建。
- `INSERT INTO visits () VALUES ()`：插入一行，字段使用默认值。
- `commit()`：确认并保存数据修改。
- `fetchone()[0]`：取出查询结果第一行的第一个值。
- `finally`：即使发生异常也关闭数据库连接。

---

## 六、Docker Compose 多服务编排

Compose 使用一个 `compose.yaml` 统一描述：

- 应用服务
- 数据库服务
- 网络
- 端口
- 环境变量
- Volume
- 启动依赖

### `.env`

```text
MYSQL_ROOT_PASSWORD=root-demo-password
MYSQL_DATABASE=docker_demo
MYSQL_USER=appuser
MYSQL_PASSWORD=app-demo-password
```

真实项目应把 `.env` 加入 `.gitignore`，避免提交密码。

### `compose.yaml`

```yaml
services:
  db:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 20

  app:
    build: .
    ports:
      - "8003:5000"
    environment:
      DB_HOST: db
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
      DB_NAME: ${MYSQL_DATABASE}
    depends_on:
      db:
        condition: service_healthy

volumes:
  mysql_data:
```

### 为什么 `DB_HOST` 是 `db`

```yaml
services:
  db:
```

Compose 网络使用服务名进行名称解析，因此应用通过：

```text
db:3306
```

访问 MySQL。

Compose 默认创建项目网络，并将 `services` 下的服务加入该网络，不需要手动执行 `docker network create`。

### 健康检查与启动依赖

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 5s
  timeout: 3s
  retries: 20
```

容器进程启动不代表 MySQL 已经可以连接。健康检查确认 MySQL 可用。

```yaml
depends_on:
  db:
    condition: service_healthy
```

表示等待 `db` 健康后再启动 `app`。

---

## 七、Docker Compose 常用命令

验证配置：

```powershell
docker compose config #类似打印compose.yaml内容
```

构建:

```powershell
docker compose build
```

启动：

```powershell
docker compose up #前台启动
docker compose up -d #后台启动
```

构建并启动：

```powershell
docker compose up -d --build
```

查看状态和日志：

```powershell
docker compose ps
docker compose logs
docker compose logs -f
docker compose logs -f app
docker compose logs -f db
```

进入服务容器：

```powershell
docker compose exec app sh
docker compose exec db mysql -uappuser -p
```

停止和恢复：

```powershell
docker compose stop
docker compose start
```

删除容器和项目网络，保留 Volume：

```powershell
docker compose down
```

连同 Volume 一起删除：

```powershell
docker compose down -v
```

区别：

```text
docker compose down
→ 删除容器和项目网络
→ 保留数据库 Volume

docker compose down -v
→ 删除容器、项目网络和 Volume
→ 数据库数据被删除
```

修改被 Dockerfile `COPY` 的文件后重新构建：

```powershell
docker compose up -d --build
```

只重新构建应用服务：

```powershell
docker compose up -d --build app
```

`--build` 的原因：宿主机代码变化不会自动修改已经构建好的镜像。

---

## 八、Compose 配置常见错误

### 缩进错误

错误信息：

```text
services.volumes additional properties 'mysql_data' not allowed
```

原因：顶层 `volumes:` 被错误缩进到了 `services:` 中。

正确层级：

```yaml
services:
  db:
    # ...
  app:
    # ...

volumes:
  mysql_data:
```

`services:` 和最下面的 `volumes:` 必须左对齐。YAML 使用空格缩进，不使用 Tab。

### 字段拼写错误

错误：

```yaml
images: mysql:8.4
enviroment:
```

正确：

```yaml
image: mysql:8.4
environment:
```

遇到以下错误时，优先检查拼写和缩进：

```text
additional properties '某字段' not allowed
```

修改后先验证：

```powershell
docker compose config
```

---

## 九、复习速记

```text
容器间通信：同一 Docker 网络 + 容器名或服务名
宿主机访问容器：使用 -p 宿主机端口:容器端口
数据库持久化：Volume名称:/var/lib/mysql
运行时配置：使用环境变量
Compose 服务互访：使用 services 下的服务名称
修改镜像内代码：重新 build
down：保留 Volume
down -v：删除 Volume 和数据
```

第三、四课的完整关系：

```text
Docker Compose
├── app 服务
│   ├── Dockerfile 构建
│   ├── 8003:5000
│   └── DB_HOST=db
├── db 服务
│   ├── mysql:8.4
│   └── /var/lib/mysql
├── 默认项目网络
│   └── app → db:3306
└── mysql_data Volume
    └── 持久化数据库文件
```

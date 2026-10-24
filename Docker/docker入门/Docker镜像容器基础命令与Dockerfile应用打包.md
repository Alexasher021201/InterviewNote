# Docker 学习笔记：第一课与第二课

## 一、核心概念

### 镜像（Image）

- 镜像是只读模板，包含程序、依赖和默认配置。
- 类似“安装包”，但更准确地说，它是创建容器的模板。
- 一个镜像可以创建多个容器。

### 容器（Container）

- 容器是镜像创建出来的运行实例。
- 应用运行在隔离的容器环境中，并没有直接安装到 Windows 中。
- 删除容器不会删除镜像。

```text
镜像（模板）
    ↓ docker run
容器（运行实例）
```

### 镜像仓库（Registry）

- 用于保存和分发镜像。
- Docker Hub 是常用的公共镜像仓库。

---

## 二、第一课：运行 Nginx 容器

### 1. 下载镜像

```powershell
docker pull nginx
```

查看本地镜像：

```powershell
docker images
```

### 2. 创建并启动容器

```powershell
docker run -d --name my-nginx -p 8080:80 nginx
```

参数含义：

| 参数 | 作用 |
|---|---|
| `docker run` | 根据镜像创建并启动新容器 |
| `-d` | 在后台运行容器 |
| `--name my-nginx` | 指定容器名称 |
| `-p 8080:80` | 将宿主机 8080 端口映射到容器 80 端口 |
| `nginx` | 使用 Nginx 镜像 |

访问地址：

```text
http://localhost:8080
```

端口映射格式：

```text
宿主机端口:容器端口
```

请求过程：

```text
浏览器 → 宿主机 8080 端口 → 容器 80 端口 → Nginx
```

> 易错点：`8080:80` 表示宿主机端口到容器端口，不表示 Windows Client 和 Linux Server。

### 3. 查看容器

查看正在运行的容器：

```powershell
docker ps
```

查看所有容器，包括已经停止的容器：

```powershell
docker ps -a
```

### 4. 查看日志

```powershell
docker logs my-nginx
```

持续查看日志：

```powershell
docker logs -f my-nginx
```

使用 `Ctrl+C` 退出日志查看，不会停止容器。

### 5. 进入容器

```powershell
docker exec -it my-nginx sh
```

退出容器终端：

```sh
exit
```

`exit` 只退出终端，不会停止容器。

### 6. 停止、启动和删除容器

```powershell
docker stop my-nginx
docker start my-nginx
docker rm my-nginx
```

强制停止并删除容器：

```powershell
docker rm -f my-nginx
```

命令区别：

| 命令 | 含义 |
|---|---|
| `docker run` | 创建一个新容器并启动 |
| `docker stop` | 停止容器，但保留容器 |
| `docker start` | 重新启动一个已有的停止状态容器 |
| `docker exec` | 在正在运行的容器中执行命令 |
| `docker rm` | 删除容器 |

> 易错点：`docker start` 不是进入容器。进入容器使用 `docker exec -it 容器名 sh`。

---

## 三、第二课：使用 Dockerfile 打包 Python Web 应用

### 1. 项目结构

```text
docker-web-demo/
├── app.py
├── requirements.txt
└── Dockerfile
```

`Dockerfile` 没有文件扩展名。

### 2. Web 应用

`app.py`：

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def home():
    return "<h1>Hello Docker!</h1>"


@app.route("/about")
def about():
    return {
        "message": "正在学习 Docker",
        "lesson": 2
    }


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`host="0.0.0.0"` 允许从容器外部访问服务。如果只监听 `127.0.0.1`，服务通常只能在容器内部访问。

### 3. Python 依赖

`requirements.txt`：

```text
Flask>=3.0,<4.0
```

### 4. Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

指令含义：

| 指令 | 作用 |
|---|---|
| `FROM` | 指定基础镜像 |
| `WORKDIR` | 设置镜像中的工作目录 |
| `COPY` | 将宿主机文件复制到镜像中 |
| `RUN` | 构建镜像时执行命令 |
| `EXPOSE` | 说明应用预计监听的容器端口 |
| `CMD` | 容器启动时执行的默认命令 |

重点区别：

```text
RUN：docker build 时执行
CMD：容器启动时执行
```

`EXPOSE 5000` 主要用于说明端口，不会自动建立宿主机端口映射。端口映射仍需使用 `docker run -p`。

### 5. 构建镜像

```powershell
docker build -t my-python-app:v1 .
```

说明：

- `-t`：设置镜像名称和标签。
- `my-python-app`：镜像名称。
- `v1`：镜像版本标签。
- 最后的 `.`：使用当前目录作为构建上下文，并从中寻找 Dockerfile 和需要复制的文件。

### 6. 运行应用

```powershell
docker run -d --name python-web -p 8000:5000 my-python-app:v1
```

访问：

```text
http://localhost:8000
http://localhost:8000/about
```

端口关系：

```text
宿主机 8000 端口 → 容器 5000 端口 → Flask
```

### 7. 修改代码并发布新版本

修改 `app.py` 后，旧容器不会自动更新，因为它使用的是此前已经构建好的镜像内容。

重新构建：

```powershell
docker build -t my-python-app:v2 .
```

删除旧容器并运行新版本：

```powershell
docker rm -f python-web
docker run -d --name python-web -p 8000:5000 my-python-app:v2
```

也可以保留旧容器，用不同名称和端口同时运行新版本：

```powershell
docker run -d --name python-web-v2 -p 8001:5000 my-python-app:v2
```

此时：

```text
旧版本：http://localhost:8000
新版本：http://localhost:8001
```

---

## 四、完整工作流程

```text
编写应用代码和 Dockerfile
          ↓
docker build 构建镜像
          ↓
docker run 创建并启动容器
          ↓
宿主机端口映射到容器端口
          ↓
浏览器访问应用
```

代码修改后的流程：

```text
修改代码
   ↓
重新构建镜像
   ↓
使用新镜像创建新容器
   ↓
运行并验证新版本
```

---

## 五、复习速查

```powershell
# 镜像
docker pull nginx
docker images
docker build -t my-python-app:v1 .
docker rmi 镜像名

# 容器
docker run -d --name 容器名 -p 宿主机端口:容器端口 镜像名
docker ps
docker ps -a
docker stop 容器名
docker start 容器名
docker rm 容器名
docker rm -f 容器名

# 调试
docker logs 容器名
docker logs -f 容器名
docker exec -it 容器名 sh
```

## 六、一句话记忆

- 镜像是模板，容器是运行实例。
- `docker run` 等于创建新容器并启动。
- `docker start` 用于重新启动已有容器。
- `docker exec` 用于在运行中的容器内执行命令。
- `宿主机端口:容器端口` 是 Docker 的端口映射格式。
- 修改被复制进镜像的代码后，需要重新构建镜像并创建容器。
- `RUN` 在构建镜像时执行，`CMD` 在容器启动时执行。

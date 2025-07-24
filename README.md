# docker-quick-reference

# 🐳 Docker 使用文档（完整版）

![Docker Logo](https://img.icons8.com/color/48/docker.png)

---

## 📌 目录

* [1. 常用 Docker 指令](#1-常用-docker-指令)
* [2. Docker 镜像源配置](#2-docker-镜像源配置)
* [3. 镜像与容器操作](#3-镜像与容器操作)
* [4. 文件挂载、端口映射与 GPU](#4-文件挂载端口映射与-gpu)
* [5. Dockerfile 模板](#5-dockerfile-模板)
* [6. Docker Compose 模板](#6-docker-compose-模板)
* [7. 镜像导入导出](#7-镜像导入导出)
* [8. 容器提交为镜像](#8-容器提交为镜像)
* [9. 附：镜像切分合并](#9-附镜像切分合并)

---

## 1. 常用 Docker 指令

| 功能     | 完整指令                      | 参数解释                               |
| -------- | ----------------------------- | -------------------------------------- |
| 查看镜像 | `docker images`               | 列出本地已有镜像                       |
| 查看容器 | `docker ps -a`                | 显示所有容器（包括已停止的）           |
| 启动容器 | `docker start 容器名/ID`      | 启动已有但停止的容器                   |
| 停止容器 | `docker stop 容器名/ID`       | 停止正在运行的容器                     |
| 删除容器 | `docker rm 容器名/ID`         | 删除已停止的容器                       |
| 删除镜像 | `docker rmi 镜像名/ID`        | 删除本地镜像                           |
| 进入容器 | `docker exec -it 容器名 bash` | 进入容器终端交互模式 (`-it`: 交互终端) |
| 容器日志 | `docker logs -f 容器名`       | 实时查看容器日志（`-f` 为持续输出）    |

---

## 2. Docker 镜像源配置（需要root权限）

**修改 Docker 配置文件**

```bash
sudo vim /etc/docker/daemon.json
```

**添加内容如下：**

```json
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}
```

**重启 Docker 生效：**

```bash
sudo systemctl daemon-reexec   # 重新载入配置
sudo systemctl restart docker  # 重启 Docker 服务
```

**临时使用docker镜像源:**

```
https://docker.aityp.com/
```






---

## 3. 镜像与容器操作

### 3.1 拉取镜像

```bash
docker pull nginx:1.25.3-alpine
```

* `nginx`: 镜像名
* `1.25.3-alpine`: 版本 + 构建方式（alpine 为体积小）

```bash
docker pull registry.example.com/redis:7.0.12
```

* 从私有仓库拉取 Redis 镜像

---

### 3.2 创建并运行容器（含端口映射、挂载）

```bash
docker run -d \
  --name webserver \
  -p 8080:80 \
  -v /path/config:/etc/nginx \
  nginx:1.25.3-alpine
```

* `-d`：后台运行
* `--name`：容器命名
* `-p`：主机8080端口映射到容器80端口
* `-v`：挂载本地配置文件目录到容器内

---

### 3.3 使用 GPU 启动容器（NVIDIA支持）

```bash
docker run --gpus all -it --rm pytorch/pytorch:latest bash
```

* `--gpus all`: 使用所有 GPU
* `--rm`: 容器退出后自动删除
* `-it`: 交互终端
* `bash`: 启动 bash 终端

---

## 4. 文件挂载、端口映射 与 GPU 示例

```bash
docker run -d \
  --name myapp \
  -p 5000:5000 \
  -v $(pwd)/app:/app \
  --gpus all \
  myimage:latest
```

* `-p 5000:5000`：主机 → 容器端口映射
* `-v $(pwd)/app:/app`：挂载当前目录 `app` 到容器 `/app`
* `--gpus all`：使用所有 GPU

---

## 5. Dockerfile 模板

```dockerfile
FROM python:3.9-slim

LABEL maintainer="you@example.com"
ENV TZ=Asia/Shanghai

COPY requirements.txt /tmp/
RUN apt-get update && \
    apt-get install -y gcc python3-dev language-pack-zh-hans && \
    locale-gen zh_CN.UTF-8 && \
    pip install -U pip -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install -r /tmp/requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

ENV LANG=zh_CN.UTF-8 \
    LANGUAGE=zh_CN:zh:en_US:en \
    LC_ALL=zh_CN.UTF-8

WORKDIR /app
COPY . /app

CMD ["python", "app.py"]
```

---

## 6. Docker Compose 模板

```yaml
version: '3.8'

services:
  web:
    image: nginx:1.25.3-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
    restart: always

  redis:
    image: registry.example.com/redis:7.0.12
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
    restart: always

  app:
    build: .
    volumes:
      - .:/app
    ports:
      - "5000:5000"
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    command: ["python", "app.py"]
```

---

## 7. 镜像导入导出

### 导出容器（不含历史）

```bash
docker export 容器ID > container.tar
```

### 导入为新镜像

```bash
docker import - new_image_name:tag < container.tar
```

---

### 保存镜像（含历史）

```bash
docker save 镜像ID > image.tar
```

### 加载镜像

```bash
docker load < image.tar
```

* `docker load` 会恢复完整历史与 tag

---

## 8. 容器提交为新镜像

```bash
docker commit -a "作者" -m "说明" 容器ID 新镜像名:tag
```

* `-a`：作者名
* `-m`：说明文字

示例：

```bash
docker commit -a "Hanyuzhe" -m "添加中文环境" 38e4c8c123a4 mynginx:with-locale
```

---

## 9. 附：镜像切分合并（大文件传输）

### 切分大镜像

```bash
split -b 7G -d -a 2 image.tar image_part.
```

* `-b 7G`: 每段最大 7GB
* `-d`: 数字编号
* `-a 2`: 后缀位数为 2

### 合并镜像

```bash
cat image_part.* > image_full.tar
```

---
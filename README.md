# 🐳 《Docker 快速参考手册》

```
```

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
* [10. docker部署vllm服务 ](#10-docker部署vllm服务)
* [附录](#更多详细教程)
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

可临时使用指令去拉取镜像，比如`轩辕镜像`源拉取：

```bash
docker pull docker.xuanyuan.me/mysql:5.7
```

注意：这里的 mysql:5.7 请替换成你需要的镜像和版本

更多镜像源参考：[dongyubin/DockerHub: 2025年7月更新，目前国内可用Docker镜像源汇总，DockerHub国内镜像加速列表，🚀DockerHub镜像加速器](https://github.com/dongyubin/DockerHub)

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

## 9. 镜像切分合并（大文件传输）

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


## 10. docker部署vllm服务
### 下载模型权重
国内使用Hugging Face下载较慢，所以采用modelscope，网速比较快。下载[deepseek](https://www.modelscope.cn/models/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B/summary)下载。
```python
from modelscope import snapshot_download

model_dir = snapshot_download(
    'deepseek-ai/DeepSeek-R1-Distill-Qwen-7B',
    cache_dir='/your/path/',  # 改成你本地目录
    revision='master',
    ignore_patterns=['*.pt', '*.bin']  # 如果你希望跳过非 safetensors 的权重
)

print(f"模型下载完成，保存路径: {model_dir}")
```
### 拉取vllm镜像
拉取临时的镜像源的镜像
```bash
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/vllm/vllm-openai:v0.9.2
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/vllm/vllm-openai:v0.9.2  docker.io/vllm/vllm-openai:v0.9.2
```
### docker部署

```bash
docker run --runtime=nvidia --gpus '"device=0"' \
        --name model_name \
        -e OPENAI_API_KEY=api_key \
        -v /your/path/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B:/deepseek \
        -p 8000:8000 \
        --ipc=host \
        vllm/vllm-openai:v0.9.1 \
        --model deepseek \
        --max-model-len 4096 \
        --tensor-parallel-size 1 \
        --gpu-memory-utilization 0.5

```
 参数说明（vLLM 常用参数）,[更多详细参数见](https://zhuanlan.zhihu.com/p/1916898243423500022)

| 参数                         | 含义                                         |
| -------------------------- | ------------------------------------------ |
| `--model`                  | 模型路径，支持 HuggingFace、本地目录                   |
| `--tokenizer`              | 若与模型不同，可显式指定 tokenizer 路径                  |
| `--max-model-len`          | 模型最大输入长度                                   |
| `--gpu-memory-utilization` | GPU 显存使用率（0.0\~1.0）                        |
| `--tensor-parallel-size`   | 多 GPU 时模型切分并行度（1 表示单卡）                     |
| `--dtype`                  | 指定精度：`auto`/`float16`/`bfloat16`/`float32` |
| `--swap-space`             | 内存不足时用于 offload 的 CPU 空间（MB）               |
| `--max-num-seqs`           | 同时生成的最大并发序列数量                              |

### 调用方式

openAI风格调用
```python3
import openai

openai.api_key = "api_key"
openai.base_url = "http://localhost:8000/v1"

response = openai.ChatCompletion.create(
    model="deepseek",  # 你可以随便写，vLLM 只部署了一个模型
    messages=[
        {"role": "user", "content": "你好，Qwen。"}
    ],
    temperature=0.7,
    max_tokens=256
)

print(response["choices"][0]["message"]["content"])
```

langchain风格调用
```python3
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage

llm = ChatOpenAI(
    base_url="http://localhost:8003/v1",
    api_key="api_key",
    model_name="deepseek",  # 形式要求，可忽略
)

response = llm([HumanMessage(content="你好，LangChain 使用你了。")])
# 或者 response = llm.invoke("你好，LangChain 使用你了。")
print(response.content)

```

## 更多详细教程

1. [yeasy/docker\_practice: Learn and understand Docker&Container technologies, with real DevOps practice!](https://github.com/yeasy/docker_practice)
2. [Docker 教程 | 菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)
3. [Windows系统安装WSL，并安装docker服务\_wsl安装docker-CSDN博客](https://blog.csdn.net/xiaohuaidan007/article/details/130160286)
4. [使用 WSL 2 安装 Docker - Anand's Blog](https://okcody.com/posts/linux/8)

---

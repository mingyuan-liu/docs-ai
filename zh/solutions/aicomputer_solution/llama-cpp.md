<!--
 * Copyright 2022-2023 SPACEMIT. All rights reserved.
 * Use of this source code is governed by a BSD-style license
 * that can be found in the LICENSE file.
 *
 * @Author: David(qiang.fu@spacemit.com)
 * @Date: 2026-02-28 14:50:46
 * @LastEditTime: 2026-03-13 13:48:53
 * @FilePath: \doc\docs-bianbu\zh\ai\llama-cpp.md
 * @Description:
-->

---
sidebar_position: 2
---

# llama.cpp

![](../static/llama-cpp-icon.png)

**llama.cpp** 是一个用纯 C/C++ 写的开源推理框架，专门让 Llama 等 GGUF/GGML 格式的大语言模型能在本地 CPU/GPU（笔记本、手机、树莓派甚至浏览器）快速运行，而无需依赖重量级框架。([https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp))

## 平台支持情况

|      平台 & 系统       |       是否支持加速      |
|-----------------------|-----------------------|
| K1 Buildroot          | ✅ 支持                |
| K1 OpenHarmony5.0     | ❌ 不支持              |
| K3 Bianbu LXQT/GNOME  | ✅ 支持                |

## Bianbu LXQT/GNOME环境使用说明

### 安装

打开终端，执行如下命令安装llama.cpp

```bash
sudo apt update
sudo apt install llama.cpp-tools-spacemit
```

**注意**：一些旧平台或者旧固件没有llama.cpp-tools-spacemit包，可以尝试

```bash
sudo apt update
sudo apt install llama-server
```

### 下载模型

目前支持5种量化格式的模型加速，下载gguf格式模型的时候，主要选择这几种量化格式的模型：
- Q4_K_M
- Q4_0
- Q4_1
- Q2_K
- Q3_K

根据芯片平台的算力下载合适参数的模型，K1平台推荐Qwen3-0.6B，下载方法如下：

```bash
wget https://www.modelscope.cn/models/unsloth/Qwen3-0.6B-GGUF/resolve/master/Qwen3-0.6B-Q4_0.gguf -P ~/
```

K3平台推荐Qwen3-30B-A3B，下载方法如下：

```bash
wget https://www.modelscope.cn/models/unsloth/Qwen3-30B-A3B-Instruct-2507-GGUF/resolve/master/Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -P ~/
```

### 使用

有3种常见的使用方法
- llama-bench
- llama-cli
- llama-server

下面以K3平台为例，分别介绍

#### llama-bench


```bash
llama-bench -m Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 -p 64 -n 64 -mmp 0 -fa 1
```
参数说明：
- -t: 指定运行测试时使用的线程数量
- -p: 指定‌提示词（Prompt）的长度‌，单位为 token
- -n: 指定输出生成长度
- -mmp: 是否启用 Multi-Modal Prompt 支持
- -fa: 是否启用 ‌Flash Attention‌ 功能

输出结果如下：

![](../static/llama-bench.png)

#### llama-cli

```bash
llama-cli -m Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 --no-mmap -c 15360
```

参数说明：
- -m: 指定.gguf格式模型文件的路径
- -t: 指定运行测试时使用的线程数量
- --no-mmap: 禁用内存映射（memory mapping）功能
- -c: 设置‌上下文长度（context size）

输出结果如下：
![](../static/llama-cli.png)

#### llama-server

启动后台llama-server服务：

```bash
llama-server -m Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 --host 127.0.0.1 --port 8080 --ctx-size 15360 --n-gpu-layers 0 --batch-size 512 --metrics --no-mmap &
```

参数说明：
- -m: 指定.gguf格式模型文件的路径
- -t: 指定运行测试时使用的线程数量
- --host: 指定服务器监听的 IP 地址
- --port: 设置服务器监听的端口号，默认为 8080
- --ctx-size: 控制模型上下文长度（以 token 为单位），影响模型处理长文本的能力
- --n-gpu-layers: 指定将模型的多少层卸载到 GPU 上运行以提升推理速度
- --batch-size: 控制一次处理的 token 数量，影响吞吐量和显存使用
- --metrics: 启用 Prometheus 格式的性能监控指标端点 /metrics，便于系统监控和性能分析
- --no-mmap: 禁用内存映射（memory mapping）功能
- -c: 设置‌上下文长度（context size）

##### 本地API请求

```bash
curl -X POST http://127.0.0.1:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "Qwen3-30B",
        "messages": [
          { "role": "user", "content": "介绍珠海？" }
        ]
      }'
```

输出结果如下：
![](../static/llama-server-api.png)


##### 浏览器请求

在浏览器中搜索 `http://localhost:8080` 打开 llama 服务器，直接在浏览器中使用 llama.cpp

![](../static/llama-server-chrome.png)

## Buildroot环境使用说明

### 安装

#### 下载

- 从[这里](https://archive.spacemit.com/spacemit-ai/llama.cpp/)下载进迭官方发布的llama.cpp，并通过scp等方式拷贝到设备上
- 通过wget命令下载（后面以0.0.5版本为例）
```bash
wget http://archive.spacemit.com/spacemit-ai/llama.cpp/spacemit-llama.cpp.riscv64.0.0.5.tar.gz
```
#### 解压

```bash
tar -zxvf spacemit-llama.cpp.riscv64.0.0.5.tar.gz
```

#### 设置LD_LIBRARY_PATH

```bash
export LD_LIBRARY_PATH=/root/spacemit-llama.cpp.riscv64.0.0.5/lib/
```

### 下载模型

根据芯片平台的算力下载合适参数的模型，K1平台推荐Qwen3-0.6B，下载方法如下：

```bash
wget https://www.modelscope.cn/models/unsloth/Qwen3-0.6B-GGUF/resolve/master/Qwen3-0.6B-Q4_0.gguf -P ~/
```

K3平台推荐Qwen3-30B-A3B，下载方法如下：

```bash
wget https://www.modelscope.cn/models/unsloth/Qwen3-30B-A3B-Instruct-2507-GGUF/resolve/master/Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -P ~/
```

**注意**：如果Buildroot中的wget不支持https路径文件的下载，报（wget: not an http or ftp url:），需要单独下载后通过scp等方式拷贝到设备中

### 使用

有3种常见的使用方法
- llama-bench
- llama-cli
- llama-server

下面以K3平台为例，分别介绍

#### llama-bench


```bash
cd spacemit-llama.cpp.riscv64.0.0.5
./bin/llama-bench -m ../Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 -p 64 -n 64
```

输出结果如下：

![](../static/llama-bench-buildroot.png)

#### llama-cli

```bash
cd spacemit-llama.cpp.riscv64.0.0.5
./bin/llama-cli -m ../Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 --no-mmap -c 15360
```

输出结果如下：
![](../static/llama-cli-buildroot.png)

#### llama-server

启动后台llama-server服务：

```bash
./bin/llama-server -m ../Qwen3-30B-A3B-Instruct-2507-Q4_0.gguf -t 8 --host 127.0.0.1 --port 8080 --ctx-size 15360 --n-gpu-layers 0 --batch-size 512 --metrics --no-mmap &
```

##### 本地API请求

```bash
curl -X POST http://127.0.0.1:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "Qwen3-30B",
        "messages": [
          { "role": "user", "content": "introduce zhuhai？" }
        ]
      }'
```

**注意**：Buildroot未安装curl，暂未验证

##### 浏览器请求

Buildroot中无浏览器，暂未验证

## FAQ

### 是否支持VLM模型？

开发中，请稍等待

### 如何确定跑模型的时候用到了AI硬件加速？

可以通过**htop**命令确认，如果跑模型的时候，CPU的8-15核有大量占用，说明用到了AI硬件加速

### llama-cli的TPS为何比llama-bench和llama-server低一些？

llama-cli在运行过程中，除了模型推理，还做了终端输出等功能，影响了TPS，并且实际项目中都是用llama-server，以llama-server为准

### llama-server一般跑几个线程？

如果对性能有要求，可以将所有的AI核跑满（K3跑8个，K1跑4个），如果需要跑多个llama-server，可以根据不同模型的算力要求分配合适的线程数量
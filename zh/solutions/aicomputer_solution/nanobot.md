<!--
 * Copyright 2022-2023 SPACEMIT. All rights reserved.
 * Use of this source code is governed by a BSD-style license
 * that can be found in the LICENSE file.
 *
 * @Author: David(qiang.fu@spacemit.com)
 * @Date: 2026-03-16 17:09:25
 * @LastEditTime: 2026-03-18 20:43:43
 * @FilePath: \doc\docs-bianbu\zh\ai\nanobot.md
 * @Description:
-->
---
sidebar_position: 6
---

# nanobot
Nanobot 是香港大学数据智能实验室开源的超轻量级个人 AI 助手，仅约 4000 行代码完整复刻了OpenClaw 智能体的核心功能。Nanobot具备网页搜索、文件操作、定时任务、记忆机制等能力，支持 24 小时实时行情分析、全栈开发、日程管理和个人知识库等场景。相比原版 43 万行代码，Nanobot 用 99% 的代码精简实现同等生产力，开发者几小时即可通读源码，快速理解 AI 调用工具与管理记忆的底层逻辑，是学习和定制 Agent 的理想选择。

## 平台支持情况

|      平台 & 系统       |       是否支持加速      |
|-----------------------|-----------------------|
| K1 Buildroot          | ❌ 不支持              |
| K1 OpenHarmony5.0     | ❌ 不支持              |
| K3 Bianbu LXQT/GNOME  | ✅ 支持                |

## 1. 安装

### 1.1. 安装依赖

安装基本依赖：
```bash
sudo apt update
sudo apt install python3-venv libffi-dev libssl-dev pkg-config
```

安装rust：
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable
source ~/.cargo/env
rustc --version
```

有如下打印说明安装成功：
![](../static/rust-install.png)

### 1.2. 下载nanobot源码

```bash
git clone https://github.com/HKUDS/nanobot.git
```

### 1.3. 编译

```bash
cd nanobot
python3 -m venv ./venv
source venv/bin/activate
pip install -e . -i https://pypi.tuna.tsinghua.edu.cn/simple
```

安装完成后，提示未配置config.json
![](../static/nanobot-use.png)

执行nanobot onboard,进行基本配置
![](../static/nanobot-onboard.png)

再查看status
![](../static/nanobot-status.png)

开启Agent，需要配置API Key
![](../static/nanobot-agent.png)

## 2. 配置
按照格式要求配置~/.nanobot/config.json中的API Key

## 3. 使用

TBD

## 4. 举一个例子

TBD
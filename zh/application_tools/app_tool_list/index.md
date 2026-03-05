sidebar_position: 1

# 应用软件栈与开发工具

## 1.1. llama.cpp

**llama.cpp** 是一个用纯 C/C++ 写的开源推理框架，专门让 Llama 等 GGUF/GGML 格式的大语言模型能在本地 CPU/GPU（笔记本、手机、树莓派甚至浏览器）快速运行，而无需依赖重量级框架。
[llama.cpp 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/bianbu/ai/llama-cpp.md)

## 1.2. Ollama

Ollama 是一款开源跨平台大语言模型（LLM）本地化部署工具，专注于简化 LLM 在本地环境中的运行、管理和推理流程。它支持用户通过简单命令在个人设备（如 PC、边缘服务器）上直接部署和调用预训练模型（如 LLaMA、DeepSeek 等），无需依赖云端服务或高性能 GPU。  
[Ollama 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/bianbu/ai/ollama.md)

## 1.3. localAI

LocalAI是用于在本地运行AI模型的完整AI堆栈。它的设计是简单、高效和可访问的，提供一个兼容OpenAI的API，允许用户在消费级硬件（包括CPU环境）上运行大型语言模型（LLM）、图像生成、语音转录等AI任务，同时保持用户的数据隐私和安全。  
[localAI 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/bianbu/ai/localai.md)

## 1.4. openwebUI  

Open-WebUI（原 Ollama WebUI）是一个开源、可自托管的 Web 管理工具，专为本地或私有化部署的大型语言模型（LLM）设计。其核心目标是提供与 ChatGPT 类似的交互体验，同时支持离线运行和高度定制化功能。  
[openwebUI 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/bianbu/ai/openwebui.md)

## 1.5. Node-RED  

Node-RED 是一个 基于流程（Flow-based）的可视化编程工具，最早由 IBM 的工程师开发，现在由开源社区（JS 基金会支持）维护。它主要用于 事件驱动的应用集成、AI应用开发、API 调度和自动化。  
[Node-RED 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/ros/02_System_configuration/2.6_Docker_Usage/2.6.2_Node-RED_Usage.md)

## 1.6. Demo Docker

Demo Docker是一个集成了spacemiT AI Demo 的容器，它以 WEB 的方式方便快捷的展示了AI调用结果，让你无需编程即可体检AI的魅力。  
[Demo Docker 使用详情](https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/ros/02_System_configuration/2.6_Docker_Usage/2.6.4_DemoZoo_Usage.md)

# 2. 开发工具

## 2.1. 模型转换
主要介绍一些常用的模型转换方法以及工具的使用，面向基础学习者。
[模型转换详细说明](./model_list/model_conversion.md)

## 2.2. 模型量化
主要介绍模型量化的方法以及工具的使用，面向基础学习者。
[模型量化详细说明](./model_list/model_quantization.md)

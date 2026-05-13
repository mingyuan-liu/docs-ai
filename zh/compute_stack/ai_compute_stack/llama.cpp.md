---
sidebar_position: 5
---

# Llama.cpp

![](../images/llama-cpp-icon.png)

> **llama.cpp** 是一个轻量级大模型推理框架，核心面向 GGUF/GGML 模型的本地推理场景。在 SpacemiT RISC-V 平台上，可以通过 RVV、IME 等硬件能力对 CPU 推理路径进行优化，并可选集成 SMT 视觉扩展以支持多模态场景。([https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp))

---

- [Llama.cpp](#llamacpp)
    - [简介](#简介)
    - [资源获取](#资源获取)
    - [文本推理 Demo](#文本推理-demo)
    - [本地构建](#本地构建)
    - [QEMU 模拟运行](#qemu-模拟运行)
    - [SMT 多模态扩展](#smt-多模态扩展)
    - [模型性能数据](#模型性能数据)


### 简介

llama.cpp 主要用于在端侧设备上运行大语言模型和多模态模型，具有以下特点：

- 原生支持 GGUF 模型格式，适合部署量化后的 LLM。
- 可通过多线程 CPU 推理快速验证模型可用性与吞吐。
- 在 SpacemiT RISC-V 平台上可启用 RVV 以及 SpacemiT 专用优化开关，以获得更高性能。
- 在开启 SMT 扩展后，可与 SpacemiT ONNXRuntime 组件协同，支持视觉编码等多模态流程。

### 资源获取

> &#x2139;&#xfe0f;llama.cpp 所需模型与工具链建议从以下目录获取，目录内容会按版本持续更新。

~~~ bash
# 预编译的llama.cpp包
wget https://archive.spacemit.com/spacemit-ai/llama.cpp/spacemit-llama.cpp.riscv64.0.0.7.tar.gz

# RISC-V 交叉编译工具链
wget https://archive.spacemit.com/toolchain/spacemit-toolchain-linux-glibc-x86_64-v1.1.2.tar.xz

# QEMU 模拟器
wget https://archive.spacemit.com/spacemit-ai/qemu/jdsk-qemu-v0.0.14.tar.gz
~~~

模型文件可使用社区公开的 GGUF 模型，或直接准备已经量化好的 GGUF 文件，例如 `Qwen2.5-0.5B-Instruct-Q4_0.gguf`。

### 文本推理 Demo

完成构建后，可以直接使用 `llama-cli` 做一个最小文本推理验证：

~~~ bash
export LD_LIBRARY_PATH=${PWD}/build/installed/lib:${LD_LIBRARY_PATH}

./build/bin/llama-cli \
  -m ./models/Qwen2.5-0.5B-Instruct-Q4_0.gguf \
  -p "介绍一下 SpacemiT RISC-V 平台。" \
  -t 8 -c 16384 --no-mmap -ub 128 --warmup
~~~

常用参数说明：

- `-m`：指定 GGUF 模型路径
- `-t`：指定推理线程数
- `-p`：指定输入提示词

### 本地构建

> &#x2139;&#xfe0f;SpacemiT RISC-V 平台构建时，建议开启 `GGML_CPU_RISCV64_SPACEMIT` 选项以启用相关优化。

~~~ bash
export RISCV_ROOT_PATH=/path/to/spacemit-toolchain-linux-glibc-x86_64-v1.1.2

cmake -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DGGML_CPU_RISCV64_SPACEMIT=ON \
    -DGGML_CPU_REPACK=OFF \
    -DLLAMA_OPENSSL=OFF \
    -DGGML_RVV=ON \
    -DGGML_RV_ZVFH=ON \
    -DGGML_RV_ZFH=ON \
    -DGGML_RV_ZICBOP=ON \
    -DGGML_RV_ZIHINTPAUSE=ON \
    -DGGML_RV_ZBA=ON \
    -DCMAKE_TOOLCHAIN_FILE=${PWD}/cmake/riscv64-spacemit-linux-gnu-gcc.cmake \
    -DCMAKE_INSTALL_PREFIX=build/installed

cmake --build build --parallel $(nproc) --config Release

pushd build
make install
popd
~~~

### QEMU 模拟运行

如果当前主机不是 RISC-V 环境，可以使用 QEMU 先做功能验证：

~~~ bash
export QEMU_ROOT_PATH=/path/to/jdsk-qemu-v0.0.14
export RISCV_ROOT_PATH_IME1=/path/to/spacemit-toolchain-linux-glibc-x86_64-v1.1.2

${QEMU_ROOT_PATH}/bin/qemu-riscv64 \
  -L ${RISCV_ROOT_PATH_IME1}/sysroot \
  -cpu max,vlen=256,elen=64,vext_spec=v1.0 \
  ${PWD}/build/bin/llama-cli \
  -m ${PWD}/models/Qwen2.5-0.5B-Instruct-Q4_0.gguf \
  -t 1 \
  -p "Hello"
~~~

### SMT 多模态扩展

如果需要在 `llama-server` 或 `llama-mtmd-cli` 中启用 SpacemiT SMT 多模态扩展，需要额外准备一个 `SPACEMIT_ORT_DIR` 目录，其中至少包含：

- `include/`
- `lib/`
- `samples/`

构建时增加以下定义：

~~~ bash
export SPACEMIT_ORT_DIR=/path/to/spacemit-ort
export LD_LIBRARY_PATH=${SPACEMIT_ORT_DIR}/lib:${LD_LIBRARY_PATH}

cmake -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DGGML_CPU_RISCV64_SPACEMIT=ON \
    -DGGML_CPU_REPACK=ON \
    -DLLAMA_OPENSSL=OFF \
    -DGGML_RVV=ON \
    -DGGML_RV_ZVFH=ON \
    -DGGML_RV_ZFH=ON \
    -DGGML_RV_ZICBOP=ON \
    -DGGML_RV_ZIHINTPAUSE=ON \
    -DGGML_RV_ZBA=ON \
    -DCMAKE_TOOLCHAIN_FILE=${PWD}/cmake/riscv64-spacemit-linux-gnu-gcc.cmake \
    -DCMAKE_INSTALL_PREFIX=build/installed \
    -DLLAMA_SERVER_SMT_VISION=ON \
    -DSPACEMIT_ORT_DIR=${SPACEMIT_ORT_DIR}
~~~

运行 `llama-server` 时需要额外传入 SMT 配置目录：

~~~ bash
export LD_LIBRARY_PATH=/path/to/spacemit-ort/lib:./build/installed/lib:${LD_LIBRARY_PATH}

./build/bin/llama-server \
  -m /path/to/model.gguf \
  --vision-backend smt \
  --smt-config-dir /path/to/smt-config-dir \
  -t 8 -c 16384 --no-mmap -ub 128 --warmup
~~~

> &#x2139;&#xfe0f;`--smt-config-dir` 目录下通常需要包含 `config.json` 以及对应的视觉 ONNX 模型文件。

### [模型性能数据](./modelzoo.md)
> 通过llama-bench测试获得
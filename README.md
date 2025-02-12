Rk3588 platform uses NPU to run LLM  models, rknn-llm server https://github.com/airockchip/rknn-llm
https://github.com/airockchip/rknn-llm


# 1. Introduction to RKLLM
RKLLM can help users quickly deploy LLM models to Rockchip chips. This repository currently supports the chip: rk3588. The overall framework is as follows:：

![Framework](https://github.com/airockchip/rknn-llm/raw/main/res/framework.jpg)

To use RKNPU, users need to first run the RKLLM-Toolkit tool on the computer, convert the trained model into a model in RKLLM format, and then use the RKLLM C API for inference on the development board.

RKLLM-Toolkit is a software development kit for users to perform model conversion and quantization on a PC.
RKLLM Runtime provides C/C++ programming interface for Rockchip NPU platform, helping users to deploy RKLLM models and accelerate the implementation of LLM applications.
The RKNPU kernel driver is responsible for interacting with the NPU hardware. It has been open sourced and can be found in the Rockchip kernel code.

## Supported Platforms
- RK3588

## Currently supported models
  - [X] [TinyLLAMA 1.1B](https://huggingface.co/TinyLlama/TinyLlama-1.1B-Chat-v1.0/tree/fe8a4ea1ffedaf415f4da2f062534de366a451e6) 
  - [X] [Qwen 1.8B](https://huggingface.co/Qwen/Qwen-1_8B-Chat/tree/1d0f68de57b88cfde81f3c3e537f24464d889081)
  - [X] [Qwen2 0.5B](https://huggingface.co/Qwen/Qwen1.5-0.5B/tree/8f445e3628f3500ee69f24e1303c9f10f5342a39)
  - [X] [Phi-2 2.7B](https://hf-mirror.com/microsoft/phi-2/tree/834565c23f9b28b96ccbeabe614dd906b6db551a)
  - [X] [Phi-3 3.8B](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct/tree/291e9e30e38030c23497afa30f3af1f104837aa6)
  - [X] [ChatGLM3 6B](https://huggingface.co/THUDM/chatglm3-6b/tree/103caa40027ebfd8450289ca2f278eac4ff26405)
  - [X] [Gemma 2B](https://huggingface.co/google/gemma-2b-it/tree/de144fb2268dee1066f515465df532c05e699d48)
  - [X] [InternLM2 1.8B](https://huggingface.co/internlm/internlm2-chat-1_8b/tree/ecccbb5c87079ad84e5788baa55dd6e21a9c614d)
  - [X] [MiniCPM 2B](https://huggingface.co/openbmb/MiniCPM-2B-sft-bf16/tree/79fbb1db171e6d8bf77cdb0a94076a43003abd9e)

# 2. Model Conversion (RKLLM-Toolkit Container Conversion Tool)
To use RKNPU, users need to first run the RKLLM-Toolkit container conversion tool on an x86 workstation to convert the trained model to an RKLLM format model, and then use the RKLLM C API for inference on the development board.

## 1. docker-compose.yml 
~~~ docker
services:
  rk3588_llm:
    image: thanhtantran/rk3588_llm
    platform: linux/amd64
    container_name: rk3588_llm
    restart: unless-stopped
    privileged: true
    volumes:
      - ./model:/root/ws
    stdin_open: true  # -i
    tty: true         # -t
    command: /bin/bash
~~~
## 2. Run
~~~ liunx
docker-compose up -d
~~~
## 3. Tải model về
Lưu vào ./model 

## 4. Download and convert the python program to ./model
~~~ liunx
wget https://raw.githubusercontent.com/airockchip/rknn-llm/main/rkllm-toolkit/examples/huggingface/test.py
~~~

## 5. Modify the model path in test.py

`modelpath = '/root/ws/Qwen2.5-3B-Instruct'`

...

## 6. Modify the name and path of the generated conversion model in test.py

`ret = llm.export_rkllm("./Qwen2.5-3B.rkllm") `

Generate Qwen2.5-3B.rkllm in the current directory (./model)

## 7. Conversion Model
### Vào container ：
~~~ liunx
 docker exec -it rk3588_llm /bin/bash
~~~
### Đến thư mục chứa model
~~~ liunx
cd /root/ws
~~~
### Chạy
~~~ liunx
python3 test.py
~~~


# 3、RKNPU driver for RK3588

Càng cao càng tốt, hiện tại là 0..9.8：
~~~ liunx
# Lệnh để xem rknpu version
cat /sys/kernel/debug/rknpu/version
# Trả về
RKNPU driver: v0.9.8
~~~

# 4、Deploy LLM server docker-compose.yml on rk3588 development board
tạo file docker-compose.yml
~~~ docker
services:
  rkllm_server:
    image: jsntwdj/rkllm_chat:1.0.1
    container_name: rkllm_chat
    restart: unless-stopped
    privileged: true
    devices:
      - /dev:/dev
    volumes:
      - ./model:/rkllm_server/model  # rkllm模型文件目录
    ports:
      - "8080:8080" # 端口自行修改
    command: >
      sh -c "python3 gradio_server.py --target_platform rk3588 --rkllm_model_path /rkllm_server/model/Qwen2.5-3B.rkllm" #  rkllm模型文件名称自行修改

~~~

Chạy
~~~ liunx
docker-compose up -d
~~~
Xem kết quả trên giao diện web `http://ip:8080`

# 5. rkllm_api_demo
## 1. rkllm_api_demo 

# 5、Hỗ trợ

Forum：https://forum.orangepi.vn
Mua mạch：https://orangepi.vn
Buy Orange Pi board: https://orangepi.net

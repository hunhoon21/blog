안녕하세요, 오늘은 Meta 에서 제공하는 하나의 GPU로 올릴 수 없는 llama 2 LLM Model을 Multi GPU 환경에서 추론 서버위에 올려보는 예제를 준비했습니다. Multi GPU 를 추론 환경으로 활용하기 위해 TnesorRT-LLM Backend로 띄운 triton inference server를 띄웁니다.

* Work Directory 는 `/home/ubuntu/workspace/` 를 기본으로 합니다.
* NVIDIA V100 GPU를 사용하기 위해 AWS EC2 에서의 `p3.8xlarge` 인스턴스를 실험 환경으로 사용합니다. 인스턴스 타입은 [여기](https://aws.amazon.com/ko/ec2/instance-types/)서 확인하실 수 있습니다.
* 주요 Reference
  * [TensorRT-LLM llama 예제](https://github.com/NVIDIA/TensorRT-LLM/tree/main/examples/llama)
  * [tensorrtllm_backend Repo](https://github.com/triton-inference-server/tensorrtllm_backend)


이번 포스트는 아래 순서대로 진행합니다.


* [LLM 모델 가중치 가져오기](#LLM-모델-가중치-가져오기)
* [HuggingFace 의 transformer 와 호환되는 가중치 파일로 convert](#HuggingFace-의-transformers-와-호환되는-가중치-파일로-convert)
* [Build TensorRT engine](#Build-TensorRT-engine)
* [triton inference server 모델 디렉토리로 archive](#triton-inference-server-모델-디렉토리로-archive)
* [tensorrtllm_backend 을 백엔드로 하는 triton inference server launch](#tensorrtllm_backend-을-백엔드로-하는-triton-inference-server-launch)
* [추론 확인](#추론-확인)


## LLM 모델 가중치 가져오기

이번 예제에서는 Meta 에서 제공하는 `llama 2` 모델을 활용합니다. 모델 가중치를 다운로드 받기 위해 우선 [여기](https://llama.meta.com/llama-downloads/) 페이지에서 키를 신청합니다. 키 신청 후 아래 커맨드를 통해 모델 가중치를 다운로드 합니다.

```shell
git clone https://github.com/meta-llama/llama.git
cd llama/
./download.sh
```

그러면 `/home/ubuntu/workspace/llama/llama-2-7b/` 디렉토리에 원하는 가중치 파일들이 다운로드 받아져 있습니다. 같이 다운로드 받아진 tokenizer 관련 파일들도 해당 디렉토리로 이동시켜줍니다.

## HuggingFace 의 transformers 와 호환되는 가중치 파일로 convert

이제 HuggingFace `transformers` 에서 실행 가능한 형태로 가중치 파일을 변환하여야 합니다. 이는 `transformers` [Repository](https://github.com/huggingface/transformers) 에서 기능을 제공합니다.

```shell
git clone https://github.com/huggingface/transformers.git
pip install transformers sentencepiece accelerate
python transformers/src/transformers/models/llama/convert_llama_weights_to_hf.py --input_dir /home/ubuntu/workspace/llama/llama-2-7b --model_size 7B --llama_version 2 --output_dir /home/ubuntu/workspace/llama-to-hf
```

위 결과로 `bin` 형식의 가중치 파일들을 얻어낼 수 있습니다.


## Build TensorRT engine
NVIDIA 에서 제공하는 [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) 을 통해 NVIDIA GPU 위에서 LLM 모델을 효율적으로 실행할 수 있는 `TensorRT Engine`을 빌드할 수 있습니다. 위에서 얻어낸 가중치 파일을 이용해서 `TensorRT Engine`을 빌드합니다. `tensorrt_llm` 패키지 버전은 `v0.8.0` 을 사용합니다. 

이 과정은 huggingface checkpoint(가중치) 를 `TensorRT-LLM` 에서 읽을 수 있는 형태로 변환하고, `TensorRT Engine` 을 빌드하는 과정을 포함합니다.

```shell
git clone https://github.com/NVIDIA/TensorRT-LLM.git
pip install "tensorrt_llm==0.8.0"  --extra-index-url https://pypi.nvidia.com
cd TensorRT-LLM/
git checkout v0.8.0
cd ..

# 가중치 변환
python /home/ubuntu/workspace/TensorRT-LLM/examples/llama/convert_checkpoint.py --model_dir /home/ubuntu/workspace/llama-to-hf/ --output_dir /home/ubuntu/workspace/tllm_checkpoint_2gpu_tp2 --dtype float16 --tp_size 2

# 엔진 빌드
trtllm-build --checkpoint_dir /home/ubuntu/workspace/tllm_checkpoint_2gpu_tp2 --output_dir /home/ubuntu/workspace/llama-trt/7B/trt-engines/fp16/2-gpu/ --gemm_plugin float16
```

아래와 같은 엔진 결과물을 얻어낼 수 있습니다.

> <img src="https://github.com/hunhoon21/hunhoon21.github.io/assets/36983960/d9783e40-c6f1-4ee2-b2c0-5fb508f923d2" alt="tensorRT LLM Engine" width="100%"/>

참고로, 엔진 결과물이 제대로 작동하는지는 아래 명령어로 확인해볼 수 있습니다. 명령어를 통해 엔진을 요약해볼 수 있습니다.

```shell
pip install absl-py rouge_score nltk
mpirun -n 2 --allow-run-as-root python ../summarize.py --test_trt_llm --hf_model_dir /home/ubuntu/workspace/llama-to-hf/ --data_type fp16 --engine_dir /home/ubuntu/workspace/llama-trt/7B/trt-engines/fp16/2-gpu/
```

## triton inference server 모델 디렉토리로 archive

이제 triton inference server 를 띄우기 위해 `triton_model_repo` 디렉토리를 구성하여야 합니다. 아래 명령어를 통해 구성할 수 있습니다.

```shell
git clone https://github.com/triton-inference-server/tensorrtllm_backend.git
cd tensorrtllm_backend
cp -r /home/ubuntu/workspace/llama-to-hf/ ./tensorrt_llm
mkdir triton_model_repo
# Copy the example models to the model repository
cp -r all_models/inflight_batcher_llm/* triton_model_repo/
# Copy the TRT engine to triton_model_repo/tensorrt_llm/1/
cp /home/ubuntu/workspace/llama-trt/7B/trt-engines/fp16/2-gpu/* triton_model_repo/tensorrt_llm/1
```

구성한 `triton_model_repo` 디렉토리 내 각 모델 디렉토리에는 `config.pbtxt` 가 존재합니다. 모든 `config.pbtxt` 에 실제 값을 넣어주어야 합니다. [여기](https://github.com/triton-inference-server/tensorrtllm_backend) 페이지에서 각 config 의 값을 참고하여 작성해주세요. `$` 값 대신 실제 값을 작성합니다.

## tensorrtllm_backend 을 백엔드로 하는 triton inference server launch
이제 디렉토리가 완성됐으면 triton inference server를 띄울 준비가 완료되었습니다. 아래 명령어로 triton inference server 를 실행하는 컨테이너를 생성하고 실행합니다.
참고로 현재까지 `TensorRT-LLM` v0.8.0 에 대응하는 triton 이미지는 [`24.02`](https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/rel-24-02.html#rel-24-02) 와 [`24.03`](https://docs.nvidia.com/deeplearning/triton-inference-server/release-notes/rel-24-03.html#rel-24-03) 입니다. 

```shell
docker run --rm -it --net host --shm-size=2g --ulimit memlock=-1 --ulimit stack=67108864 --gpus all -v /home/ubuntu/workspace/tensorrtllm_backend:/tensorrtllm_backend nvcr.io/nvidia/tritonserver:24.02-trtllm-python-py3 bash

# IN CONTAINER!
python3 -m pip install sentencepiece protobuf
cd /tensorrtllm_backend/
python3 scripts/launch_triton_server.py --world_size=2 --model_repo=/tensorrtllm_backend/triton_model_repo
```

## 추론 확인
이제 추론 요청을 보내면 응답을 확인할 수 있습니다.

> <img src="https://github.com/hunhoon21/hunhoon21.github.io/assets/36983960/5323cada-a887-4ab2-a0b8-eca8331e6843" alt="tensorRT LLM Engine" width="100%"/>

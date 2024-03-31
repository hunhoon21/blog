안녕하세요, 오늘은 Meta 에서 제공하는 하나의 GPU로 올릴 수 없는 llama 2 LLM Model을 Multi GPU 환경에서 추론 서버위에 올려보는 예제를 준비했습니다. Multi GPU 를 추론 환경으로 활용하기 위해 TnesorRT-LLM Backend로 띄운 triton inference server를 띄웁니다.

* Work Directory 는 `/home/ubuntu/workspace/` 를 기본으로 합니다.
* NVIDIA V100 GPU를 사용하기 위해 AWS EC2 에서의 `p3.8xlarge` 인스턴스를 실험 환경으로 사용합니다. 인스턴스 타입은 [여기](https://aws.amazon.com/ko/ec2/instance-types/)서 확인하실 수 있습니다.
* 주요 Reference
  * [TensorRT-LLM llama 예제](https://github.com/NVIDIA/TensorRT-LLM/tree/main/examples/llama)
  * [tensorrtllm_backend Repo](https://github.com/triton-inference-server/tensorrtllm_backend)


이번 포스트는 아래 순서대로 진행합니다.


* [LLM 모델 가중치 가져오기](#CKA-란?)
* [HuggingFace 의 transformer 와 호환되는 가중치 파일로 convert 하기](#시험-준비-과정과-합격-후기)
  * [시험 준비 과정](#시험-준비-과정)
  * [합격 후기](#합격-후기)
* [Build TensorRT engine](#꿀팁)
* [triton inference server 모델 디렉토리로 archive 하기](#결론)
* [tensorrtllm_backend 을 백엔드로 하는 triton server launch](#문제-유형과-정리-노트-공유)
* [추론 확인](#추론-확인)



## LLM 모델 가중치 가져오기

이번 예제에서는 Meta 에서 제공하는 `llama 2` 모델을 활용합니다. 모델 가중치를 다운로드 받기 위해 우선 [여기](https://llama.meta.com/llama-downloads/) 페이지에서 키를 신청합니다. 키 신청 후 아래 커맨드를 통해 모델 가중치를 다운로드 합니다.

```shell
git clone https://github.com/meta-llama/llama.git
cd llama/
./download.sh
```

그러면 `/home/ubuntu/workspace/llama/llama-2-7b/` 디렉토리에 원하는 가중치 파일들이 다운로드 받아져 있습니다. 같이 다운로드 받아진 tokenizer 관련 파일들도 해당 디렉토리로 이동시켜줍니다.

## HuggingFace 의 transformers 와 호환되는 가중치 파일로 convert 하기

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

## triton inference server 모델 디렉토리로 archive 하기


## tensorrtllm_backend 을 백엔드로 하는 triton server launch

## 추론 확인


위 가중치 파일을 TensorRT LLM 엔진에서 실행할 수 있는 형태로 convert 하기 a.k.a  

위 과정은 checkpoint(가중치) 를 trt_engine 에서 읽을 수 있는 형태로 변환하고, engine을 빌드하는 과정으로 이뤄져있음.

추가적으로 빌드한 engine 과 huggingface model 가중치로 추론이 되는지 확인하는 절차가 필요함

3번에서 구한 파일로 

4번에서 얻은 디렉토리로  하기


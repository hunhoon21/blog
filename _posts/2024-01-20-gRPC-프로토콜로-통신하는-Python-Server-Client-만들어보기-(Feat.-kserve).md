안녕하세요. 오늘은 gRPC 프로토콜로 통신하는 Server와 Client를 만들어보려고 합니다.
직접 만들어보면서 gRPC 프로토콜 서비스를 구축하는 것에 대해 이해도가 높아지기를 기대합니다.

* [gRPC 프로토콜이란?](#gRPC-프로토콜이란?)
* [gRPC 프로토콜로 통신을 위해 필요한 준비물](#gRPC-프로토콜로-통신을-위해-필요한-준비물)
  * [proto file](#proto-file)
  * [Protocol Buffer Compiler](#Protocol-Buffer-Compiler)
  * [gRPC Server](#gRPC-Server)
  * [gRPC Client](#gRPC-Client)
* [gRPC Server / Client 작성하기](#gRPC-Server-/-Client-작성하기)
  * [proto file 작성](#proto-file-작성)
  * [proto file 컴파일](#proto-file-컴파일)
  * [gRPC Server 작성하기](#gRPC-Server-작성하기)
  * [gRPC Client 작성하기](#gRPC-Client-작성하기)
* [결론](#결론)

## gRPC 프로토콜이란?
gRPC는 Google에서 개발한 고성능, 오픈 소스 원격 프로시저 호출 (RPC) 프레임워크입니다. 이 프로토콜은 서버와 클라이언트 간의 효율적인 통신을 가능하게 하는 현대적인, 강력한 방법을 제공합니다. gRPC는 HTTP/2를 기반으로 작동하며, 이를 통해 다중 요청 및 응답이 동시에 하나의 TCP 연결을 통해 전송될 수 있어 네트워크 사용 효율이 증가합니다.

gRPC의 핵심 구성 요소 중 하나는 Protocol Buffers (protobuf)입니다. 이는 구조화된 데이터를 직렬화하는 경량이면서 빠른 메커니즘을 제공합니다. .proto 파일에 데이터 구조와 서비스를 정의하면, Protocol Buffer Compiler (protoc)를 사용하여 특정 프로그래밍 언어로 소스 코드를 자동 생성할 수 있습니다. 이러한 접근 방식은 개발자가 다양한 언어와 플랫폼에서 쉽게 gRPC 서비스를 구축하고 사용할 수 있도록 지원합니다.

`kserve` 에서는 모델을 배포할 때 gRPC 프로토콜을 지원합니다. kserve 에서 정의한 proto 파일을 통해 gRPC 프로토콜을 사용하는 Server / Client 구축의 실제 예시를 살펴보려고 합니다.

## gRPC 프로토콜로 통신을 위해 필요한 준비물
gRPC 프로토콜을 사용하여 서버-클라이언트 통신 시스템을 구축하기 위해서는 `proto file`, `Protocol Buffer Compiler`, `gRPC Server`, `gRPC Client` 가 필요합니다.

### proto file
`.proto` 파일은 서비스 인터페이스와 서비스에서 주고 받는 메시지 형식을 정의하는 데 사용됩니다. 프로토콜 버퍼(Protocol Buffers, protobuf)는 gRPC에서 사용하는 기본 메시지 교환 형식을 의미합니다. 이 파일에는 서비스의 RPC 메소드, 요청 및 응답 메시지의 구조가 정의되어 있으며, 이는 통신 중에 교환되는 데이터의 청사진 역할을 합니다. 이 파일은 언어에 독립적이며, 다양한 프로그래밍 언어로 손쉽게 변환될 수 있습니다. 아래 예제에서 파이썬 언어로 컴파일 되는 것을 확인할 수 있습니다.

### Protocol Buffer Compiler
`.proto` 파일을 작성한 후, 이를 각각의 프로그래밍 언어에 적합한 형태로 변환해야 합니다. 이를 위해 Protocol Buffer Compiler, 즉 protoc가 사용됩니다. protoc는 .proto 파일을 입력으로 받아, 서버와 클라이언트가 이해할 수 있는 특정 언어의 소스 코드로 변환하는 역할을 합니다. 변환된 소스코드를 이용하여 각 언어별로 Server 와 Client를 작성할 수 있습니다.
### gRPC Server
gRPC 서버는 클라이언트의 요청을 받고 처리하는 역할을 합니다. 클라이언트로부터 RPC 호출을 받으면, 해당 요청을 처리하고 필요한 데이터를 응답으로 반환합니다. gRPC 서버는 다양한 언어로 구현될 수 있으며, `.proto` 파일에서 정의된 서비스 인터페이스를 구현해야 합니다. gRPC 서버는 HTTP/2 프로토콜을 사용하여 통신하며, 고성능과 비동기 처리를 지원합니다.

### gRPC Client
gRPC 클라이언트는 서버에 연결하여 RPC 호출을 수행하는 역할을 합니다. 클라이언트는 `.proto` 파일에 정의된 메소드를 호출하여 서버에 요청을 보내고, 서버로부터 응답을 받습니다. 클라이언트는 서버와 동일한 프로토콜 버퍼 메시지 형식을 사용하여 통신하며, 다양한 프로그래밍 언어로 작성될 수 있습니다. 아래 에제에서는 파이썬으로 gRPC Client를 작성합니다.


## gRPC Server / Client 작성하기
### proto file 작성
proto file 작성
proto file
`.proto` 파일은 gRPC 인터페이스와 메시지의 형식을 정의하는 데 사용됩니다. 이 파일은 서비스의 메서드와 메시지 구조를 명시적으로 정의하며, gRPC 시스템의 기본 설계도 역할을 합니다. gRPC 프로토콜을 사용하여 서버와 클라이언트 간의 통신을 위한 메시지 형식과 서비스 메소드를 이 파일에서 구체적으로 정의합니다.
Kserve에서 제공하는 예제를 참고했습니다.

```go
syntax = "proto3";
package inference;
```

여기에서는 gRPC 프로토콜의 버전과 패키지 이름을 지정합니다. proto3는 프로토콜 버퍼 언어의 최신 버전을 나타내며, 이는 gRPC에서 권장하는 표준입니다. package inference는 이 .proto 파일 내에서 정의된 모든 서비스와 메시지가 inference라는 네임스페이스 아래에 있음을 나타냅니다.

```go
// Inference Server GRPC endpoints.
service GRPCInferenceService
{
  rpc ServerLive(ServerLiveRequest) returns (ServerLiveResponse) {}
  rpc ModelInfer(ModelInferRequest) returns (ModelInferResponse) {}
}
```

메서드를 정의한 부분입니다. 여기서는 GRPCInferenceService라는 서비스를 정의하고 있으며, 이 서비스는 두 가지 RPC 메소드, 즉 ServerLive와 ModelInfer를 제공합니다. 각 메소드는 요청 및 응답 메시지 유형을 명시합니다. 이를 통해 클라이언트가 서버에 특정 작업을 요청할 수 있는 인터페이스가 구성됩니다.

```go
message ServerLiveRequest {}

message ServerLiveResponse
{
  // True if the inference server is live, false if not live.
  bool live = 1;
}
```

ServerLive 메서드에서 주고받는 메시지를 정의한 부분입니다. ServerLiveRequest 메시지는 추가적인 입력 필드 없이 구성되어 있어, 단순한 상태 확인 요청을 나타냅니다. ServerLiveResponse 메시지는 서버의 상태를 나타내는 live라는 불리언 필드 하나를 포함합니다. 이는 클라이언트가 서버의 상태를 확인할 때 서버가 '살아있는(live)' 상태인지를 반환합니다. 이러한 방식으로 gRPC는 다양한 종류의 요청 및 응답 메시지를 간단하고 효율적으로 정의할 수 있습니다.

```go
message ModelInferRequest
{
  // Model inference request details
  string model_name = 1;             // The name of the model to be used for inference
  string model_version = 2;          // The version of the model
  string id = 3;                     // A unique identifier for the request
  map<string, InferParameter> parameters = 4; // Additional inference parameters
  repeated InferInputTensor inputs = 5;       // The input tensors for inference
  repeated InferRequestedOutputTensor outputs = 6; // The output tensors that are requested
  repeated bytes raw_input_contents = 7;      // Raw input data in binary format
  
  ...
}
```
모델 추론을 위한 ModelInfer 메서드의 입력 메시지를 정의한 부분입니다. 이 메시지는 모델 추론에 필요한 다양한 정보를 담고 있습니다. model_name과 model_version 필드는 사용할 모델의 이름과 버전을 지정합니다. id 필드는 각 요청을 구별하기 위한 고유 식별자 역할을 합니다. parameters는 추가적인 추론 매개변수를 포함하며, 이는 키-값 쌍의 맵 형태로 표현됩니다. inputs와 outputs 필드는 각각 추론에 사용될 입력 텐서와 요청된 출력 텐서를 나타냅니다. 마지막으로, raw_input_contents는 바이너리 형식의 원시 입력 데이터를 포함합니다.

```go
message ModelInferRequest
{
  ...

  message InferInputTensor
  {
    string name = 1;                  // Input tensor의 이름
    string datatype = 2;              // Input tensor의 데이터 타입
    repeated int64 shape = 3;         // Input tensor의 형태
    map<string, InferParameter> parameters = 4; // 추가적인 입력 텐서 매개변수
    InferTensorContents contents = 5; // Input tensor의 내용
  }

  message InferRequestedOutputTensor
  {
    string name = 1;                  // 요청된 출력 텐서의 이름
    map<string, InferParameter> parameters = 2; // 추가적인 출력 텐서 매개변수
  }
}
```

InferInputTensor와 InferRequestedOutputTensor는 ModelInferRequest 내에 중첩된 메시지 형식으로 정의되어 있습니다. InferInputTensor는 추론에 사용될 각 입력 텐서에 대한 세부 사항을 포함하며, 이름, 데이터 타입, 형태, 추가 매개변수, 그리고 실제 텐서 내용을 정의합니다. InferRequestedOutputTensor는 요청된 출력 텐서에 대한 정보를 담고 있으며, 출력 텐서의 이름과 추가 매개변수를 포함합니다. 이러한 중첩된 메시지 구조는 gRPC에서 복잡한 데이터 구조를 효율적으로 표현하는 데 사용됩니다.

```go
message ModelInferResponse
{
  // Response structure for ModelInfer method
  message InferOutputTensor
  {
    string name = 1;                  // Output tensor의 이름
    string datatype = 2;              // Output tensor의 데이터 타입
    repeated int64 shape = 3;         // Output tensor의 형태
    map<string, InferParameter> parameters = 4; // 추가적인 출력 텐서 매개변수
    InferTensorContents contents = 5; // Output tensor의 내용
  }

  string model_name = 1;             // 추론이 수행된 모델의 이름
  string model_version = 2;          // 모델의 버전
  string id = 3;                     // 요청과 연결된 고유 식별자
  map<string, InferParameter> parameters = 4; // 추가적인 추론 매개변수
  repeated InferOutputTensor outputs = 5;      // 출력 텐서들
  repeated bytes raw_output_contents = 6;      // 바이너리 형식의 원시 출력 데이터
}
```

ModelInferResponse 메시지는 ModelInfer 메서드의 응답 결과를 정의합니다. 이 메시지는 추론 작업에 대한 결과 정보를 포함하며, 응답에는 모델의 이름, 버전, 요청 ID, 추가 매개변수, 출력 텐서들 및 원시 출력 데이터가 포함됩니다. 각 출력 텐서는 InferOutputTensor 메시지를 통해 자세하게 표현되며, 이는 이름, 데이터 타입, 형태, 추가 매개변수 및 텐서의 실제 내용을 포함합니다.

```go
message InferParameter
{
  oneof parameter_choice
  {
    bool bool_param = 1;
    int64 int64_param = 2;
    string string_param = 3;
  }
}

message InferTensorContents
{
  repeated bool bool_contents = 1;
  repeated int32 int_contents = 2;
  repeated int64 int64_contents = 3;
  repeated uint32 uint_contents = 4;
  repeated uint64 uint64_contents = 5;
  repeated float fp32_contents = 6;
  repeated double fp64_contents = 7;
  repeated bytes bytes_contents = 8;
}
```

InferParameter와 InferTensorContents 메시지는 앞서 나왔던 구조입니다. InferParameter는 다양한 유형의 매개변수(불리언, 정수, 문자열)를 나타내며, InferTensorContents는 다양한 데이터 유형을 가진 텐서 내용을 표현합니다. 이러한 유연성은 gRPC가 다양한 데이터 유형과 상황에 적합하게 활용될 수 있도록 합니다.

이제 필요한 메서드와 메시지 정의가 완료되었습니다. .proto 파일에 기재된 이 정의들을 바탕으로, Protocol Buffer Compiler를 사용하여 gRPC 서버와 클라이언트 코드를 생성할 수 있습니다. 이렇게 생성된 코드는 Python, Java 등 다양한 프로그래밍 언어에서 gRPC 서버와 클라이언트를 구현하는 데 사용될 수 있습니다. 이 과정은 gRPC 기반의 통신 시스템 개발의 핵심 단계이며, 이를 통해 효율적이고 강력한 원격 프로시저 호출 시스템을 구축할 수 있습니다.

위에서 작성한 `inference.proto` 을 정리하면 다음과 같습니다.
```go
syntax = "proto3";
package inference;

// Inference Server GRPC endpoints.
service GRPCInferenceService
{
  rpc ServerLive(ServerLiveRequest) returns (ServerLiveResponse) {}
  rpc ModelInfer(ModelInferRequest) returns (ModelInferResponse) {}
}

message ServerLiveRequest {}

message ServerLiveResponse
{
  // True if the inference server is live, false if not live.
  bool live = 1;
}

message ModelInferRequest
{
  // Model inference request details
  string model_name = 1;             // The name of the model to be used for inference
  string model_version = 2;          // The version of the model
  string id = 3;                     // A unique identifier for the request
  map<string, InferParameter> parameters = 4; // Additional inference parameters
  repeated InferInputTensor inputs = 5;       // The input tensors for inference
  repeated InferRequestedOutputTensor outputs = 6; // The output tensors that are requested
  repeated bytes raw_input_contents = 7;      // Raw input data in binary format
  
  message InferInputTensor
  {
    string name = 1;                  // Input tensor의 이름
    string datatype = 2;              // Input tensor의 데이터 타입
    repeated int64 shape = 3;         // Input tensor의 형태
    map<string, InferParameter> parameters = 4; // 추가적인 입력 텐서 매개변수
    InferTensorContents contents = 5; // Input tensor의 내용
  }

  message InferRequestedOutputTensor
  {
    string name = 1;                  // 요청된 출력 텐서의 이름
    map<string, InferParameter> parameters = 2; // 추가적인 출력 텐서 매개변수
  }
}

message ModelInferResponse
{
  string model_name = 1;             // 추론이 수행된 모델의 이름
  string model_version = 2;          // 모델의 버전
  string id = 3;                     // 요청과 연결된 고유 식별자
  map<string, InferParameter> parameters = 4; // 추가적인 추론 매개변수
  repeated InferOutputTensor outputs = 5;      // 출력 텐서들
  repeated bytes raw_output_contents = 6;      // 바이너리 형식의 원시 출력 데이터

  // Response structure for ModelInfer method
  message InferOutputTensor
  {
    string name = 1;                  // Output tensor의 이름
    string datatype = 2;              // Output tensor의 데이터 타입
    repeated int64 shape = 3;         // Output tensor의 형태
    map<string, InferParameter> parameters = 4; // 추가적인 출력 텐서 매개변수
    InferTensorContents contents = 5; // Output tensor의 내용
  }
}

message InferParameter
{
  oneof parameter_choice
  {
    bool bool_param = 1;
    int64 int64_param = 2;
    string string_param = 3;
  }
}

message InferTensorContents
{
  repeated bool bool_contents = 1;
  repeated int32 int_contents = 2;
  repeated int64 int64_contents = 3;
  repeated uint32 uint_contents = 4;
  repeated uint64 uint64_contents = 5;
  repeated float fp32_contents = 6;
  repeated double fp64_contents = 7;
  repeated bytes bytes_contents = 8;
}
```

### proto file 컴파일
`.proto` 파일을 작성한 후에 이를 특정 프로그래밍 언어에 맞는 소스 코드로 변환할 수 있습니다. 이 과정은 Protocol Buffer Compiler (protoc)를 사용하여 수행됩니다. protoc는 다양한 프로그래밍 언어를 지원하며, 각 언어에 맞는 gRPC 코드를 생성할 수 있는 플러그인을 제공합니다. [여기](https://grpc.io/docs/languages/) 에서 원하는 언어에 대한 구체적인 컴파일 정보를 파악할 수 있습니다.

위에서 작성한 `inference.proto` 파일을 컴파일하기 위한 파이썬 예제입니다. `inference.proto` 와 같은 디렉토리에서 수행하면 됩니다.

```shell
pip install grpcio-tools
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. inference.proto
```

위 명령어로 inference.proto 파일을 Python 소스 코드로 변환합니다. --python_out 옵션은 Protocol Buffers 메시지를 Python 클래스로 변환하며, --grpc_python_out 옵션은 Python gRPC 서비스 코드를 생성합니다. 다음과 같은 파일들이 생성됩니다

* `inference_pb2.py`
  * Python Protocol Buffers 클래스 파일
  * 이 파일은 .proto 파일에서 정의한 메시지 형식을 Python 클래스로 변환한 것입니다. 이 클래스는 데이터를 직렬화하고 역직렬화하는 데 사용됩니다.

* `inference_pb2_grpc.py`
  * Python gRPC 서비스 파일
  * 이 파일에는 gRPC 서비스 인터페이스와 클라이언트 스텁(stub)이 정의되어 있습니다. 서비스 인터페이스는 서버 측에서 구현되어야 하며, 클라이언트 스텁은 클라이언트가 서버의 메소드를 호출하는 데 사용됩니다.
  * 실제 코드를 보면 `GRPCInferenceServiceServicer` 클래스를 통해 서버 클래스를 작성하는 예시를 살펴볼 수 있습니다. 해당 클래스처럼 실제 Server 코드를 구현합니다.

컴파일 과정을 완료한 후에는 생성된 파일들을 사용하여 gRPC 서버와 클라이언트를 구현할 수 있습니다. 이 단계에서는 .proto 파일에 정의된 서비스 메소드를 구현하고, 클라이언트에서 이러한 메소드를 호출하는 로직을 작성합니다. 이를 통해 정의한 프로토콜에 따라 서버-클라이언트 통신 시스템을 구축할 수 있습니다.

### gRPC Server 작성하기
```python
import grpc
from concurrent import futures
import inference_pb2
import inference_pb2_grpc


class InferenceServicer(inference_pb2_grpc.GRPCInferenceServiceServicer):
    def ServerLive(self, request, context):
        response = inference_pb2.ServerLiveResponse()
        response.live = True  # 서버가 활성화 상태임을 나타냄
        return response

    def ModelInfer(self, request, context):
        # 예시로 입력 데이터를 로그로 출력
        for input_tensor in request.inputs:
            print(
                f"Received input tensor '{input_tensor.name}' with shape {input_tensor.shape}"
            )

        # 모의 추론 결과 생성
        response = inference_pb2.ModelInferResponse()
        response.model_name = request.model_name
        response.model_version = request.model_version
        response.id = request.id

        # 추론 결과를 InferOutputTensor에 설정
        output_tensor = response.outputs.add()
        output_tensor.name = "output_tensor"
        output_tensor.datatype = "FLOAT32"
        output_tensor.shape.extend([1, 10])  # 예시 shape
        output_tensor.contents.fp32_contents.extend(
            [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
        )

        return response


def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    inference_pb2_grpc.add_GRPCInferenceServiceServicer_to_server(
        InferenceServicer(), server
    )
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()


if __name__ == "__main__":
    serve()

```

gRPC 서버를 작성하는 과정은 클라이언트의 요청을 처리하고 응답을 반환하는 로직을 구현하는 것을 포함합니다. 위의 코드 예시는 Python에서 gRPC 서버를 구축하는 방법을 보여줍니다. 이 서버는 InferenceServicer 클래스를 통해 `ServerLive` 와 `ModelInfer` 두 가지 gRPC 메소드를 구현하고 있습니다.

* InferenceServicer 클래스
  * InferenceServicer 클래스는 inference_pb2_grpc.GRPCInferenceServiceServicer를 상속받아, .proto 파일에서 정의한 gRPC 서비스를 구현합니다.
각 메소드는 요청 객체(request)와 컨텍스트(context)를 인자로 받습니다.
* ServerLive 메소드
  * ServerLive 메소드는 서버의 '살아있음' 상태를 확인하는 간단한 요청을 처리합니다.
  * 이 메소드는 ServerLiveResponse 메시지를 생성하고, 서버가 활성화 상태임을 나타내는 live 필드를 True로 설정합니다.
* ModelInfer 메소드
  * ModelInfer 메소드는 모델 추론 요청을 처리합니다.
  * 입력 텐서 데이터를 로그로 출력하고, 모의 추론 결과를 생성하여 반환합니다.
  * 이 메소드는 ModelInferResponse 메시지를 생성하고, 출력 텐서에 대한 데이터를 설정합니다.
* 서버 실행
  * serve 함수는 gRPC 서버를 초기화하고 실행합니다.
  * ThreadPoolExecutor를 사용하여 비동기 처리가 가능한 워커 스레드를 생성합니다.
  * 서비스 클래스의 인스턴스를 서버에 추가하고, 특정 포트(예: 50051)에서 수신 대기합니다.
  * wait_for_termination 메소드는 서버가 종료될 때까지 대기합니다.
* 실행
  * 서버는 스크립트가 실행될 때 serve 함수를 호출하여 시작됩니다.
  * 이 gRPC 서버 예시는 실제 모델 추론 서비스의 기본 틀을 제공합니다. 실제 애플리케이션에서는 ModelInfer 메소드 내부의 로직을 실제 모델 추론 로직으로 교체하고, 입력과 출력 데이터 형식을 애플리케이션에 맞게 조정해야 합니다.

### gRPC Client 작성하기
```python
import grpc
import inference_pb2
import inference_pb2_grpc


def run():
    with grpc.insecure_channel("localhost:50051") as channel:
        stub = inference_pb2_grpc.GRPCInferenceServiceStub(channel)
        response = stub.ServerLive(inference_pb2.ServerLiveRequest())
        print("Server is live: " + str(response.live))

    with grpc.insecure_channel("localhost:50051") as channel:
        stub = inference_pb2_grpc.GRPCInferenceServiceStub(channel)

        # ModelInfer 요청
        infer_request = inference_pb2.ModelInferRequest()
        infer_request.model_name = "example_model"
        infer_request.model_version = "1.0"
        infer_request.id = "12345"

        # InferInputTensor 설정
        input_tensor = infer_request.inputs.add()
        input_tensor.name = "input_tensor"
        input_tensor.datatype = "FLOAT32"
        input_tensor.shape.extend([1, 224, 224, 3])
        input_tensor.contents.fp32_contents.extend([0.0] * (1 * 224 * 224 * 3))

        infer_response = stub.ModelInfer(infer_request)
        print("Model Infer Response:")
        print("Model Name:", infer_response.model_name)
        print("Model Version:", infer_response.model_version)
        print("Request ID:", infer_response.id)
        print("Output Tensor:", infer_response.outputs[0].name)
        print("Output Data:", infer_response.outputs[0].contents.fp32_contents)


if __name__ == "__main__":
    run()
```

gRPC 클라이언트는 서버에 데이터를 요청하고 응답을 받는 역할을 수행합니다. 위의 코드 예시는 Python에서 gRPC 클라이언트를 구현하는 방법을 보여줍니다. 이 클라이언트는 `ServerLive`와 ModelI`nfer 두 가지 gRPC 메소드를 호출합니다.

* 클라이언트 초기화
  * grpc.insecure_channel을 사용하여 서버에 연결합니다. 여기서 "localhost:50051"은 연결할 서버의 주소입니다.
  * inference_pb2_grpc.GRPCInferenceServiceStub 클래스를 통해 gRPC 서비스 스텁을 생성합니다. 이 스텁을 사용하여 서버의 메소드를 호출할 수 있습니다.
* ServerLive 메소드 호출
  * ServerLive 메소드는 서버의 상태를 확인하기 위해 호출됩니다.
  * 서버의 응답을 받아 출력하여 서버가 활성화 상태인지 확인합니다.
* ModelInfer 메소드 호출
  * ModelInfer 메소드를 호출하여 모델 추론을 요청합니다.
  * ModelInferRequest 객체를 생성하고, 필요한 정보(모델 이름, 버전, 요청 ID) 및 입력 텐서 데이터를 설정합니다.
  * 입력 텐서(input_tensor)는 FLOAT32 데이터 타입으로 설정되며, 모의 데이터를 입력으로 제공합니다.
* 응답 처리
  * 서버로부터의 응답(infer_response)을 받아 출력합니다. 이 응답에는 모델 이름, 버전, 요청 ID, 출력 텐서 정보 등이 포함됩니다.
* 실행
  * 클라이언트는 스크립트 실행 시 run 함수를 호출하여 시작되며, 두 gRPC 메소드의 호출 결과를 출력합니다.
  * 이 gRPC 클라이언트 예시는 간단한 모델 추론 서비스와 서버 상태 확인을 위한 기본적인 구현을 제공합니다. 실제 애플리케이션에서는 이 코드를 사용하여 다양한 유형의 데이터를 처리하고, 필요한 서비스 로직에 따라 클라이언트를 확장할 수 있습니다.


## 결론

이번 포스팅에서 gRPC 프로토콜을 사용하여 Python 서버와 클라이언트를 구축하는 과정을 살펴보았습니다. gRPC는 고성능, 언어 중립적인 통신 프로토콜로, 효율적인 서버-클라이언트 통신을 가능하게 합니다. 우리는 proto 파일 작성부터 시작하여, Protocol Buffer Compiler를 이용한 컴파일, 그리고 실제 gRPC 서버와 클라이언트의 구현까지 다뤘습니다.

다음 포스팅에서는 HTTP 프로토콜 및 웹서비스에 대한 내용을 다루려고 합니다.

읽어주셔서 감사합니다 :)

* [프로메테우스란?](#프로메테우스란?)
* [Exporter를 통한 메트릭 수집](#Exporter를-통한-메트릭-수집)
* [Custom Exporter 작성하기](#비동기-작동-흐름-이해하기)
  * [Custom Exporter Deployment](#비동기-작동-흐름-도식화)
  * [Custom Exporter Service](#비동기-작동-흐름-도식화)
  * [Custom Exporter Service Monitor](#비동기-작동-흐름-도식화)
* [결론](#결론)

## 프로메테우스란?

프로메테우스(Prometheus)는 오픈 소스 시스템 모니터링 및 경고 도구입니다. 이는 시계열 데이터를 수집하고 저장하기 위해 사용되며, 주로 IT 인프라 및 서비스 모니터링에 활용됩니다. Prometheus의 핵심 기능은 다양한 소스에서 메트릭(metrics)을 수집하는 것입니다. 이를 위해, 다양한 ‘exporter’들이 존재하며, 이들은 Prometheus가 이해할 수 있는 형식으로 메트릭을 제공합니다.

## Exporter를 통한 메트릭 수집
Exporter는 Prometheus가 모니터링할 수 있는 형태로 메트릭을 전달하는 역할을 합니다. 다양한 유형의 exporter가 있으며, 이들은 서버, 데이터베이스, 하드웨어 등의 메트릭을 수집합니다. 각 exporter는 /metrics 엔드포인트를 통해 데이터를 제공하며, Prometheus 서버는 이를 주기적으로 스크레이핑하여 메트릭을 수집합니다.

## Custom Exporter 작성하기
커스텀 Exporter 개발은 특정 애플리케이션이나 서비스에 맞춤화된 메트릭을 수집할 수 있게 해줍니다. 아래 단계들을 통해 커스텀 Exporter를 개발하고 Prometheus에서 수집할 수 있습니다.

Custom Exporter를 작성하고, Prometheus에서 데이터를 수집해가기 하기 위해서는 아래 3가지를 작성해야합니다.
* Custom Exporter Deployment
  * 원하는 메트릭을 API로 전달해주는 Web Application이 필요하고, Deployment 형태로 띄울 수 있습니다.
* Custom Exporter Service
  * 위에서 작성한 Custom Exporter Pod이 통신 가능하게끔 만들어주는 service가 필요합니다.
* Custom Exporter Service Monitor
  * Prometheus 에서 수집해갈 수 있도록 service monitor를 설정해야 합니다.



### Custom Exporter Deployment
Custom Exporter를 배포하기 위해서는 먼저, /metrics 엔드포인트로 메트릭을 전달하는 API 서버가 필요합니다. 예를 들어, Python으로 구현된 API 서버는 Prometheus 클라이언트 라이브러리를 사용하여 메트릭을 생성하고 제공할 수 있습니다. 이러한 서버는 Docker 이미지로 패키징되어 Kubernetes 클러스터에 배포될 수 있습니다. 

아래는 파이썬으로 random한 숫자를 메트릭으로 전달하는 API를 파이썬으로 구현한 예시입니다.

```python
import time
import random
from http.server import HTTPServer, BaseHTTPRequestHandler
from prometheus_client import Summary, Counter, Gauge, generate_latest

# Create a metric to track time spent and requests made.
REQUEST_TIME = Summary('request_processing_seconds', 'Time spent processing request')
RANDOM_NUMBER = Gauge('random_number', 'A random number')
RANDOM_NUMBER_2 = Counter('random_number_2', 'A random number 2.0')

class MetricsHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/metrics':
            self.send_response(200)
            self.send_header('Content-Type', 'text/plain; charset=utf-8')
            self.end_headers()
            self.wfile.write(generate_latest())
        else:
            self.send_response(404)
            self.end_headers()

def run_server():
    server = HTTPServer(('0.0.0.0', 9000), MetricsHandler)
    server.serve_forever()

if __name__ == "__main__":
    # Start the server in a new thread
    from threading import Thread
    server_thread = Thread(target=run_server)
    server_thread.start()

    # Update metrics in the main thread
    try:
        while True:
            RANDOM_NUMBER.set(random.randint(1, 20))
            RANDOM_NUMBER_2.inc(random.randint(1, 30))
            time.sleep(1)
    except KeyboardInterrupt:
        print("Interrupted by user, shutting down.")
        server.shutdown()

```

도커파일을 작성하여 image로 빌드합니다.
```Dockerfile
FROM python:3.8-slim

# Copy the script into the container
COPY . /app
WORKDIR /app

# Install any necessary packages
RUN pip install prometheus_client

# Expose the port your exporter runs on
EXPOSE 9000

# Command to run the script
CMD ["python", "./custom_exporter.py"]
```

위 이미지를 활용하여 Deployment yaml 을 작성합니다.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-exporter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-exporter
  template:
    metadata:
      labels:
        app: custom-exporter
    spec:
      containers:
      - name: custom-exporter
        image: <CUSTOM_IMAGE>
        ports:
        - containerPort: 9000
```

### Custom Exporter Service
Kubernetes 환경에서는 Exporter에 대한 Service를 생성하여 통신이 가능하게 합니다. 이 Service는 Exporter가 생성하는 메트릭에 대한 접근 포인트 역할을 하며, Prometheus가 메트릭을 스크레이핑할 수 있도록 설정됩니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: custom-exporter
  labels:
    app: custom-exporter
spec:
  selector:
    app: custom-exporter
  ports:
    - name: web
      protocol: TCP
      port: 9000
      targetPort: 9000
```

위 Service를 정의 한 후 포트포워딩을 통해 직접 접속해보면 metrics이 작성되고 있는 것을 확인할 수 있습니다.

> <img src="https://github.com/hunhoon21/hunhoon21.github.io/assets/36983960/39be94a5-dd44-4500-ad8d-42012b9ffaf6" alt="async-logic" width="100%"/>

### Custom Exporter Service Monitor
마지막으로, Prometheus가 이 서비스를 스크레이핑할 수 있도록 Service Monitor 리소스를 정의합니다. `matchLabels` 을 보고 해당 레이블을 가진 서비스를 모니터링 대상에 추가합니다. 이는 저희 환경에서 service monitor를 통해 프로메테우스 수집해갈 수 있도록 이미 처리되어 있기 때문에 가능합니다.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: custom-exporter-monitor
  labels:
    release: prometheus # Adjust this label to match your Prometheus Operator selector
spec:
  selector:
    matchLabels:
      app: custom-exporter # This should match labels of your service
  endpoints:
  - port: web
    interval: 30s
```

설정 후 prometheus UI 에서 `Status` → `Configuration` 을 들어가면 scrape_configs 에 기대한 custom exporter 가 추가된것을 확인할 수 있습니다.

> <img src="https://github.com/hunhoon21/hunhoon21.github.io/assets/36983960/6f0b476a-0df7-43a6-944c-b3edfd1f418e" alt="async-logic" width="100%"/>

또한, prometheus UI 에서 `Status` → `Targets` 을 들어가면 마찬가지로 수집되고 있는 serviceMonitor를 확인할 수 있습니다.

> <img src="https://github.com/hunhoon21/hunhoon21.github.io/assets/36983960/fcea0837-e9ea-4fed-a002-2ca24cbf45dc" alt="async-logic" width="100%"/>


## 결론

이러한 단계를 거쳐 커스텀 Exporter를 개발하고 배포함으로써, Prometheus는 사용자 정의 메트릭을 수집할 수 있게 됩니다. 이를 통해 특정 애플리케이션 또는 환경에 특화된 모니터링 서비스를 구축할 수 있습니다.

이 글이 커스텀 Exporter를 통해 프로메테우스로 메트릭을 수집하는 방법을 이해하는데 도움이 되었기를 기대합니다.
오늘도 읽어주셔서 감사합니다 :)

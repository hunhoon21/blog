안녕하세요. 오늘은 Client에게 데이터를 전송하는 websocket server를 간단하게 만들어보겠습니다. 해당 포스트는 [[Python] Websocket을 사용하는 방법](https://nowonbun.tistory.com/674)를 참고하였습니다.

### 예상독자
* asyncio에 대해 어느정도 아는 사람
* websocket 서버를 간단하게 구축해보고 싶은 사람

## websocket
* 서버와 클라이언트 간에 socket connection을 유지해서 양방향 통신 또는 데이터 전송이 가능하도록 하는 기술
* 양방향 통신이기에 서버에서 내부적으로 계산한 Score를 지속적으로 Client(예. 웹 브라우저)로 전송할 수 있음
* client가 server에 데이터를 전송하는 경우는 위 [[Python] Websocket을 사용하는 방법](https://nowonbun.tistory.com/674)를 참고

## 코드
Client에게 지속적으로 데이터를 전송하는 websocket 서버는 파이썬으로, websocket client는 HTML로 작성합니다.

### 웹소켓 서버
```python
import asyncio
import websockets


async def accept(websocket, path):
    i = 0
    while True:
		# send data to websocket client from this server
        print(f"sending <{i}> to client")
        await websocket.send(f"client got {i}")
        i += 1
        await asyncio.sleep(1.0)
        if i == 10:
            print("This server pasued..")
            await websocket.send("The client should refresh")
            break

# open websocket server on http://localhost:9998
websocket_server = websockets.serve(accept, "localhost", 9998)

loop = asyncio.get_event_loop()
loop.run_until_complete(websocket_server)
loop.run_forever()
```

### 웹소켓 클라이언트
```html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
  </head>
<body>
  <form>
    <!-- 접속 종료 버튼 -->
    <input onclick="disconnect()" value="Disconnect" type="button">
  </form>
  <br />
  <!-- 출력 area -->
  <textarea id="messageTextArea" rows="10" cols="50"></textarea>
  <script type="text/javascript">
    // 웹 서버를 접속한다.
    var webSocket = new WebSocket("ws://localhost:9998");
    // 웹 서버와의 통신을 주고 받은 결과를 출력할 오브젝트를 가져옵니다.
    var messageTextArea = document.getElementById("messageTextArea");
    // 소켓 접속이 되면 호출되는 함수
    webSocket.onopen = function(message){
      messageTextArea.value += "Server connect...\n";
    };
    // 소켓 접속이 끝나면 호출되는 함수
    webSocket.onclose = function(message){
      messageTextArea.value += "Server Disconnect...\n";
    };
    // 소켓 통신 중에 에러가 발생되면 호출되는 함수
    webSocket.onerror = function(message){
      messageTextArea.value += "error...\n";
    };
    // 소켓 서버로 부터 메시지가 오면 호출되는 함수.
    webSocket.onmessage = function(message){
      // 출력 area에 메시지를 표시한다.
      messageTextArea.value += "Receive From Server => "+message.data+"\n";
    };
    // 서버로 메시지를 전송하는 함수
    function sendMessage(){
      var message = document.getElementById("textMessage");
      messageTextArea.value += "Send to Server => "+message.value+"\n";
      //웹소켓으로 textMessage객체의 값을 보낸다.
      webSocket.send(message.value);
      //textMessage객체의 값 초기화
      message.value = "";
    }
    function disconnect(){
      webSocket.close();
    }
  </script>
</body>
</html>
```

### 해당 코드 실행시 도식화
> <img src="https://user-images.githubusercontent.com/36983960/116805562-47856000-ab62-11eb-80f8-fd6e40f0e944.png" alt="websocket_server" width="50%"/>

## How to use
1. 서버 실행
```
python websoket_server.py
```
2. index.html 파일 실행(웹 브라우저)
  <img src="https://user-images.githubusercontent.com/36983960/116805652-cc707980-ab62-11eb-842b-7475f5aa4305.png" alt="index.html" width="60%"/>

### 실행 화면
웹 브라우저에서 띄운 html client를 통해 서버로부터 실시간으로 데이터가 들어오는 것을 확인할 수 있습니다.
> ![websocket_server_to_client](https://user-images.githubusercontent.com/36983960/116805817-e1014180-ab63-11eb-9c5c-636226626b76.gif)
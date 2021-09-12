이 글은 Masnun 의 블로그 글 [PYTHON ASYNCIO: FUTURE, TASK AND THE EVENT LOOP](https://masnun.com/2015/11/20/python-asyncio-future-task-and-the-event-loop.html) 을 번역한 글입니다. 비동기 개념을 잡을때 개인적으로 도움이 되었습니다!

## Event Loop (이벤트 루프)
어떤 플랫폼이든 비동기적으로 무언가를 하려고 하면, Event Loop가 필요합니다. Event Loop란 실행할 작업(tasks)를 등록, 실행, 지연 혹은 취소할 수 있고, 또 이러한 작업과 관련된 다양한 Event 들을 처리할 수 있는 Loop를 의미합니다. 우리는 여러개의 비동기 함수를 이 Event Loop에 예약(schedule)합니다. 이 Loop는 한 함수를 실행하고, 이 함수가 I/O를 기다리는 동안 잠시 일시정지하고 다른 함수를 실행할 수 있습니다. 앞서 잠시 멈췄던 함수의 I/O가 끝나면 해당 함수는 재개(resume)됩니다. 즉 두개 이상의 함수를 동시적으로(co-operatively) 함께 실행할 수 있습니다. 이것이 Event Loop의 주요 목적입니다.

Event Loop는 프로세스를 처리하기 위해 resource를 복잡하게 다루는 함수를 쓰레드 풀(thread pool)에 전달할 수도 있습니다. 사실 이처럼 Event Loop의 내부는 상당히 복잡합니다. 하지만 당장 이에 대해 크게 걱정할 필요는 없습니다. Event Loop는 비동기 함수를 예약하고 실행할 수 있는 메커니즘이라는 것만 기억하면 됩니다.

## Futures / Tasks
혹시 Javascript에도 익숙한 독자라면 `Promise` 라는 개념도 알 것입니다. 파이썬에서는 비슷한 개념으로 `Future` 와 `Task` 가 있습니다. `Future` 는 미래에 어떤 결과를 가질 객체를 뜻합니다. `Task`는 코루틴(coroutine) 을 감싸고 있는 `Future` 의 subclass 입니다. 해당 코루틴이 완료되면 `Task`는 실제로 결과물을 갖게됩니다.

## Coroutines (코루틴)
[지난 포스트](https://masnun.com/2015/11/13/python-generators-coroutines-native-coroutines-and-async-await.html)에서 코루틴에 대해 다뤘습니다. 코루틴은 함수를 일시정지하고 나열된 값을 주기적으로 반환할 수 있는 방법입니다. 코루틴은 파이썬에서 `yield from` 혹은 `await` 키워드를 통해 함수 실행을 일시정지할 수 있습니다. 해당 함수는 `yield` 구문이 실제로 값을 받을때까지 멈춰져 있습니다.

## Event Loop와 Future/Task 함께 사용하기
이는 간단합니다. Event Loop를 정의하고 우리의 Future / Task 객체를 해당 Event Loop에 등록하면 됩니다. 이 Loop는 알아서 스케쥴하고 Future / Task 객체를 실행할겁니다. 또 우리는 Future / Task 객체에 콜백 함수를 추가하여 완료되었을 때 알수도 있습니다.

우리는 코루틴을 활용하는 경우가 많습니다. 코루틴을 Future로 감싸고 Task 객체를 얻습니다. 코루틴은 `yield` 하면 잠시 멈추고, 실제 값을 가지게 되었을때 재개됩니다. 반환될 때는 Task가 완료되고 값을 가진다는 의미입니다. 관련된 모든 콜백은 실행됩니다. 코루틴이 에러를 발생하면, Task는 실패된 상태로 남게됩니다.

예제 코드를 함께 보겠습니다.

```python
import asyncio
 
 
@asyncio.coroutine
def slow_operation():
    # yield from suspends execution until
    # there's some result from asyncio.sleep
 
    yield from asyncio.sleep(1)
 
    # our task is done, here's the result
    return 'Future is done!'
 
 
def got_result(future):
    print(future.result())
 
 
# Our main event loop
loop = asyncio.get_event_loop()
 
# We create a task from a coroutine
task = loop.create_task(slow_operation())
 
# Please notify us when the task is complete
task.add_done_callback(got_result)
 
# The loop will close when the task has resolved
loop.run_until_complete(task)
```
* `@asyncio.coroutine` 는 코루틴을 선언하는 부분입니다.
* `loop.create_task(slow_operation())` 는 `slow_operation()` 에 의해 만들어질 코루틴으로 부터 나온 Task를 만듭니다.
* `task.add_done_callback(got_result)` 는 해당 태스크에 콜백을 추가합니다.
* `loop.run_until_complete(task)` 는 Task가 실제 값을 가질때까지 Event Loop를 실행합니다. 실제 값을 가지면 해당 Loop는 종료됩니다.

`run_until_complete` 함수는 Loop를 관리하는 멋진 방법입니다. 물론 아래처럼도 할 수 있습니다.

```python
import asyncio
 
 
async def slow_operation():
    await asyncio.sleep(1)
    return 'Future is done!'
 
 
def got_result(future):
    print(future.result())
 
    # We have result, so let's stop
    loop.stop()
 
 
loop = asyncio.get_event_loop()
task = loop.create_task(slow_operation())
task.add_done_callback(got_result)
 
# We run forever
loop.run_forever()
```
콜백함수에서 명시적으로 loop의 종료를 하므로 forever하게 실행시켜도 기대했던대로 완료됩니다.
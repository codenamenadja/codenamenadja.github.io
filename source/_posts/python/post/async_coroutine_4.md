---
title: 비동기 코루틴 번역-4
p: python/post/async_coroutine_4
date: 2019-06-19 18:11:02
tags: ['python', 'python post']
---

## 제너레이터로_코루틴_구성하기

우리의 코루틴들은 `Asyncio`라이브러리 등으로 단순화 될 것이다.
우리는 `generator`, `future`, `yield from`을 사용할 것이다.

```python
class Future:
    def __init__(self):
        self.result = None
        self._callbacks = []

    def add_done_callback(self,fn):
        self._callbacks.append(fn)

    def set_result(self,result):
        self.result = result
        for fn in self._callbacks:
            fn(self)
```

위의 future은 원천적으로 `pending`상태이다. 그리고 `set_result`를 통해서 `resolved`된다.  
fetcher에 futures와 코루틴을 적용해 보자.

```python
class Fetcher:
    def fetch(self):
        self.sock = socket.socket()
        self.sock.setblocking(False)
        try:
            self.sock.connect({"xkcd.com", 80})
        except BlockingIOError:
            pass
        selector.register(self.sock.fileno(),
        EVENT_WRITE,
        self.connected
        )

    def connected(self, key, mask):
        print("connected!")
```

`fetch` 메서드는 소켓연결을 시도하고, 콜백을 등록한다.  
그 콜백은 소켓이 준비되었을때, 실행될 것이다.  
우리는 이 두 단계를 하나의 코루틴으로 조합할 것이다.

```python
def fetch(self):
    sock = socket.socket()
    sock.setblocking(False)
    try:
        sock.connect({"xkcd.com", 80})
    except:
        pass

    f = Future()

    def on_connected():
        f.set_result(None)

    selector.register(
        sock.fileno(),
        EVENT_WRITE,
        on_connected
        )

    yield f
    selector.unregister(
        sock.fileno()
    )
    print("connected")
```

이제 `fetch`는 제너레이터 함수이다. 우리는 `pending`중인 future을 생성한다.  
그리고 yield를 통해서 이것을 정지시키고, 소켓이 연결되었을 때까지, stop상태로 heap에서 대기한다.  
`on_connected`함수가 등록되어 커널이 소켓에서 예약해놓은 이벤트 시그널을 감지하면,  
다시 실행될 것이다. 그 콜백인 내부 on_connected가 future을 진행 시킬 것이다.  

그러면 future이 진행될 때, 무엇이 제너레이터로 재진입하게 만드는가?  
우리는 코루틴 `driver`가 필요하다. 이것을 `task`라 명명하겠다.

```python
class Task:
    def __init__(self, coro):
        self.coro = coro
        f = Future()
        f.set_result(None)
        self.step(f)

    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except StopIteration:
            return

        next_future.add_done_callback(self.step)

# begin fetching
fetcher = Fetcher("/353/")
Task(fetcher.fetch())

loop()
```

task는 fetch제너레이터를 `None`을 `send()`함으로써 시작시킨다.  
fetch는 그것이 future을 `yield`하기 전까지 동작한다.  
그 future은 task가 `next_future`로써 처리한다.  
소켓이 연결되고 나면, 이벤트 루프가 `on_connected`를 실행시킨다.  
그 함수는 future을 진행시키고, `step`을 호출한다.  
그 `step`함수가 코루틴 함수로 재진입을 실행한다.

## 코루틴을_`yield from`으로_전환하기

소켓이 연결되고 나면, 우리는 GET에 대한 응답을 받을 것이다.  
이러한 과정은 더이상 callback들 사이에 분산될 필요가 없다.  
그것을 하나의 제너레이터 함수에 모을 것이다.

```python
def fetch(self):
    sock.send(request.encode("ascii"))

    while True:
        f = Future()

        def on_readable():
            f.set_result(sock.recv(4096))

        selector.register(
            sock.fileno(),
            EVENT_READ,
            on_readable
        )

        chunk = yield f
        selector.unregister(sock.fileno())

        if chunk:
            self.response += chunk
        else:
            # Done Reading
            break
```

햔재 코드는 소켓에서 모든 메세지를 읽고 있다.  
우리가 이 fetch에서 어떻게 서브루틴으로 전환 할 수 있을까?  
`yield from`을 사용해야 할 때이다, 예시를 보자

```python
def gen_fn():
    result = yield 1
    print(f"result of yield_1 {result}")
    result_2 = yield 2
    print(f"result of yield_2 {result_2}")
    return "done"

def caller_fn():
    gen = gen_fn()
    rv = yield from gen # line 10
    print(f"return val of yield-from: {rv}")
    return "all done"

if __name__ == "__main__":
    caller = caller_fn()
    caller.send(None)
    print("instruction: ", caller.gi_frame.f_lasti)
    # intruction: 10
    caller.send("hello")
    # result of yield_1 hello
    print("instruction: ", caller.gi_frame.f_lasti)
    # instruction: 10
    caller.send("goodbye")
    # result of yield_2 goodbye
    # return val of yield-from: done
    # Traceback...
    # StopIteration: all done
```

`caller`가 `gen`으로 부터 `yield from` 하는동안, `caller`은 instruction상태를 유지한다.  
루틴으로부터 send받은 것을 yield from 으로 컨텍스트를 양도한다.  
`caller`밖의 시점에서 바라볼때, 우리는 그것이 caller에서 오는 것인지, 또 다른 제너레이터에서 오는 것인지 알 수 없다.  
그리고 `gen`내부에서, 우리는 또한, `caller`에서 온것인지, 루틴에서 온것인지 알 수 없다.

> `yield from`진술은 마찰없는 선언문이다. 값들이 `gen`이 종료되기 전까지, 안팎으로 이동한다.

코루틴은 이렇듯 `yield from`을 통해서, 컨텍스트를 서브-코루틴으로 양도하고, 그 결과를 받을 수 있다.  
마지막에 `gen`에서 돌려받은 값을 "return val of yield-from: done"으로 받은 것을 생각하자.

`rv = yield from gen`

우리가 초기에 callback기반 비동기 프로그래밍을 평가 할때, 우리의 주요 불평은, `stack ripping`에 대한 것이었다.

* `stack-ripping`: 콜백이 예외를 발생하면, `stack trace`가 무의미해진다.

그러면 우리가 만든 코루틴 기법은 어떤 결과를 보일까?

```python
def gen_fn():
    # raise Exception("my error")
    result = yield 1
    print(f"result of yield_1 {result}")
    result_2 = yield 2
    print(f"result of yield_2 {result_2}")
    return "done"
```

가장 link-depth가 깊은 스텍에 에러를 억지로 발생해 보면, 정확히 에러를 분석할 수 있는 것을 알 수 있다.  
`caller_fn`이  `gen_fn`을 바라보고 있다는 것을 스텍 트레이싱을 통해서 알 수 있다.  
우리는 서브-코루틴을 호출할때, 래핑을 하여 에러 핸들링을 할 수 있다.

```python
def caller_fn():
    gen = gen_fn()
    try:
        rv = yield from gen
    except Exception as exc:
        print(f"caught {exc}")
    print(f"return val of yield-from:  {rv}")
    return "all done"
```

따라서 우리는 서브-코루틴을 마치 일반적인 함수(서브루틴)처럼 로직은 정비하였다.  
본론으로 돌아가 우리의 fetcher에서 부터 서브코루틴을 정비하자.  
우리는 `read`코루틴을 작성하고, 하나의 chunk를 받을 것이다.

```python
def read(sock):
    f = Future()

    def on_readable():
        f.set_result(sock.recv(4096)) # self.result로 받으면 등록된 콜백을 전부 순회하고 종료한다.

    selector.register(socket.fileno(), EVENT_READ, on_readable)
    chunk = yield f # EVENT_READ가 발생하면 onreadable을 이미 등록하고, 동작하면이 아니라 바로 Future 인스턴스를 돌려준다.
    # 해당 소켓은 논 블로킹 소켓으로 4kb이상 들어오면 그만큼 읽어내고 flush
    # 모든 약속들은 Future인스턴스에 의해 처리된다고 보면 된다.
    selector.unregister(sock.fileno())
    return chunk # chunk는 읽은 데이터가 아니라, 루틴에서 전달 받은 것.

def read_all(sock):
    response = []
    chunk = yield from read(sock) # yield 1회 하고 종료 바로 돌아온다.
    while chunk: # 루틴에서 전달 받은것이 유효한 동안.
        response.append(chunk) #
        chunk = yield from read(sock)
    return b"".join(response)

# reads = read_all(sock)
# reads.send(None) f를 돌려주고, 종료 chunk에는 값이 할당되지 않는다.
# 이후 넌블로킹 소켓에서 데이터를 읽으면 예약한 대로, 4kb 청크 데이터에 대한 Future에 예약된 콜백들을 실행하지만 별도 send를 추가로 진행해야 read가 마저 진행된다.

# reads.send(True) read로 전달, read의 chunk에 처음으로 값이 맺힌다.
# 연달아 read는 1회 종료되고, unregi된다. read_all로 리턴하면서 종료한다.
# read_all의 chunk에 chunk를 리턴한다.

# Future_set_result를 통한 콜백 순회가 이루어 지고, 그러면 그 콜백의 마지막 요소가
# reads.send(True)로 다시 실행하면,
# 만약 그러다가 읽을 데이터가 이제 없다면, 소켓 selector타입아웃으로 처리해서, chunk를 False로 리턴하는 함수도 예약하면,
# 마지막에 read_all에 send(False)한 것은 b"".join(response)를 통해서 완전한 응답을 받을것이다.
# 요청이 들어온 것에 대한 하나의 처리가 종료되는 것.
```

`yield from read`는 2번의 send를 통해 종료되기 전까지 `read all`을 진행시키지 않는다.  
그리고 1회 실행에서 `read_all`이 정지되어 있는 동안, asyncio의 EVENTLOOP이 등록되어 있는게 계속 돌아간다.

```python
class Fetcher:
    def fetch(self):
        sock.send(request.encode("ascii"))
        self.response = yield from read_all(sock)
```

기적적이게도, Task class는 수정이 필요하지 않다.  

`Task(fetcher.fetch())`  
`loop()`

`read`가 future을 yield할 때, task는 이것을 yield from 진술을 통한 연결에서 전달 받는다.  
정확하게 future이 fetch에서 직접적으로 yield되는 것처럼 되는 것처럼.  
loop가 future을 진행시키면, task는 그것을 결과를 fetch로 보낸다. 그리고 값은 read에서 돌려받는다.  
마치 정확히 task가 read를 조종하는 것처럼 보인다.


| class name |           method            |                                                                    return                                                                    |
| :--------: | :-------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------: |
|    Task    |      init(self, coro)       |                                       self.coro = coro, f = Future(), f.set_result(None), self.step(f)                                       |
|    Task    |     step(self, future)      |                            next_future = self.coro.send(future.result), next_future.add_done_callback(self.step)                             |
|  Fetcher   |      init(self, path)       |                                                             self.location = path                                                             |
|  Fetcher   |         fetch(self)         | socket setup, socket.connect(path), f = Future, selector.register(socket, EVENT_WRITE, f.set_result(None)), yield f, selector.unregi(socket) |
|   Future   |         init(self)          |                                                   self.result = None, self._callbacks = []                                                   |
|   Future   | add_done_callback(self, fn) |                                                          self._callbacks.append(fn)                                                          |
|   Future   |  set_result(self, result)   |                                             self.result = result, for fn in _callbacks: fn(self)                                             |

|             from              | name                     |                             do                              |
| :---------------------------: | :----------------------- | :---------------------------------------------------------: |
|       Fetcher_instance        | Fetcher.init(self, path) |                 fetcher = Fetcher("/353/")                  |
|         Task_instance         | Task(fetcher.fetch())    |        task.coro = fetcher.fetch(), f = Future()...         |
|        Future_instance        | Future.init(self)        |          self.result = None, self._callbacks = []           |
|             task              | Task(fetcher.fetch())    |                       ...self.step(f)                       |
|             task              | step(f)                  |     next_future = self.coro.send(future.result#None)...     |
|            fetcher            | send(None)               |                     yield new Future()                      |
|             task              | step(f)                  |         ...new_future.add_done_callback(self.step)          |
| selector_EVENT_WRITE_Callback | future.set_result(None)  |                       task.step(None)                       |
|             task              | step(f)                  |                   self.coro.send(None)...                   |
|            fetcher            | send(None)               | unregister(socket), print("connected"), raise StopIteration |
|             task              | step(f)                  |              ... except StopIteration: return               |

이것으로 1회 Path에 대한 소켓 연결이 활성화 되어 WRITE가 가능해진 상태가 되었음을 공지 받는다.

우리의 코루틴 구현을 좀 더 치밀하게 만들기 위해서, 하나를 수정한다.  
우리는 future이 socket이벤트를 기다릴때, `Fetcher.fetch`에서 현재 yield를 사용해서, future인스턴스를 리턴한다.  
하지만 `yield from`을 사용해서, 서브코루틴을 가르키도록 할 것이다.  
이것은 더욱 깔끔한 것이 될 것이다. 그러면 `yield from`을 사용하는 코루틴은 더이상 어떤 타입에 대해서 기다리는지 걱정할 필요가 없게 된다.

```python
class Future:
    def __iter__(self):
        yield self
        return self.result
```

이제 next(f)는 코루틴 함수가 되었고,  
`yield f` 를 `yield from f`로 교체하도록 한다.

기능상 동일하다. 진행중인 Task는 future을 `send`를 통해서 받고, 그리고 `future`이 EVENT_WRITE되고 나면, `yield from`의 2번째가 작동할 것이다.
그렇다면, `yield from`을 사용하는 이점은 무엇인가?  
`yield`로 바로 돌려 받는 것과 `yield from`을 통해 서브코루틴을 가르키도록 하는 것은 무엇이 더 나은가?  

메서드는 `caller`의 영향 없이 자유롭게 구현을 변환 할 수 있다.  
두 경우 모두 다, caller은 오직 `yield from`메서드만을 통해서 결과를 기다릴 수 있게 된다.

우리는 어떻게 `asyncio`가 두 세상을 동시에 가지는지 그려냈다.:  
동시성 I/O가 멀티쓰레드보다 더 효과적이고, callback보다 더 정교하다.  
물론 실제 asyncio는 우리의 밑그림 보다 더욱 정교하다.  
실제 프레임웤은 `zero-copy I\O`, `fair scheduling`, `exception handling`을 요구한다.

asyncio유저에게 코루틴을 활용해 코딩하는 것은 당신이 여기서 본 것 보다 훨씬 간단하다.  
우리는 첫번째 원칙부터 코루틴을 구현해 낸것이다. 그렇게 때문에 callbacks, tasks, futures등을 본 것이다.  
그리고 Non-blocking소켓까지 보았고, select를 호출했다.  
> 그러나 `asyncio`를 사용하면 위의 것들은 전혀 등장하지 않을 것이다.  
> 우리가 위에서 그렸듯이 아주 부드럽게 URL을 fetch할 수 있다.

```python
@asyncio.coroutine
def fetch(self, url):
    response = yield from self.session.get(url)
    #session이 yield를 거쳐서 최종적으로 return 하는 것. send(something)은 session.get의 yield로 전달 된다.
    body = yield from response.read()
    # Response.read가 yield하는 것이 계속 되고 종료되면 return 하는 것을 chunk로 누적시켜 return할 것이다.
    return body
    # 최종적으로 EVENT_READ등을 통한 콜백들이 순차적으로 예약되고 진행되면서 알아서 작동하고, 마지막에 return 될 것이다.
```

이제 우리의 원래 목적인 async web crwaler를 asyncio를 통해서 구현하는 것으로 돌아간다.

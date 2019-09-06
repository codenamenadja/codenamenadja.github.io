---
title: 비동기 코루틴 번역-5-End
p: python/post/async_coroutine_5_end
date: 2019-09-05 18:17:02
tags: ["python", "python post"]
---

## 코루틴_정비하기

우리는 어떻게 크롤러가 동작하는지 설명하는 것으로 부터 시작했다.  
이제 asyncio 코루틴을 사용해서 구현할 차례이다.

1. 우리 크롤러는 첫 페이지를 fetch한다.
2. 그것의 링크들을 파싱하고,
3. 그것들을 queue에 더할 것이다.
4. 웹사이트를 돌고나면
5. pages를 동시적으로 fetching할 것이다.
    * 하지만 클라이언트와 서버의 제한된 load때문에, 우리를 최대 워커 수를 필요로 한다.
6. 언제든 워커가 페이지를 fetching하는 것이 종료되면, 그 워커는 급히 다음 링크를 queue에서 꺼내야한다.
7. 그리고 더 돌아갈 작업이 없다면, 일부 워커는 멈춰야한다.
8. 하지만 워커가 수많은 링크를 지닌 페이지를 히트한다면,
9. 큐는 급격히 커지고, 멈췄던 워커들은 다시 동작해야한다.
10. 따라서 우리의 프로그램은 모든 일이 끝났을 때만 종료된다.

워커가 쓰레드라고 가정해보자, 우리가 크롤러의 알고리즘을 어떻게 표현해야 할까?  
우리는 파이썬 표준 라이브러리에서 **Synchronized queue**를 사용할 수 있다. (공유자원 개념?)  
아이템이 queue에 들어갈 때마다, queue는 그의 tasks의 count를 증가시킨다.  
워커 쓰레드들은 task_done을 개별 아이템이 끝날 때마다 호출한다.  
그러면 메인쓰레드가 Queue에 들어간 각아이템이 task_done콜에 의해 매칭될 때까지 Queue.join에서 블록하고, 종료할 것이다.

코루틴은 asyncio queue를 이용한 동일한 패턴을 사용한다.

```python
try:
    from asyncio import JoinableQueue as Queue
except ImportError: # 파이썬 3.5 버전 이상
    from asyncio import Queue
```

우리는 워커들의 공유상태를 crwaler class에 모을 것이고, 메인 로직을 그것의 crwal메서드에 작성할 것이다.

```python
from asyncio import Queue, get_event_loop
from aiohttp import ClientSession

loop = get_event_loop()

class Crwaler:
    def __init__(self, root_url, max_redirect):
        self.max_tasks = 10
        self.max_redirect = max_redirect
        self.q = Queue()
        self.seen_urls = set()

        # aiohttp의 클라이언트 세션이 connection을 풀링하고,
        # HTTP Keep-alives for us.
        self.session = ClientSession(loop=loop)
        self.q.put((root_url, self.max_redirect))
```

현재 q에서 끝나지 않은 작업은 하나이다. 우리의 메인 스크립트로 돌아가서,  
event루프와 crwal메서드를 실행한다.

```python
@coroutine
def crawl(self):
    workers = [Task(self.work()) for _ in range(self.max_tasks)]
    yield from self.q.join()
    from w in workers:
        w.cancel()

if __name__ == "__main__":
    crawler = Crwaler("http://xkcd.com", max_redirect=10)
    loop.run_until_complete(crawler.crawl())
```

만약 워커가 쓰레드 였다면, 우리는 그들이 동시에 실행되는 것을 원하지 않을 것이다.  
그들이 특별히 필요로 해지기 전까지 그들이 생성되는 비용을 피하기 위해서,  
쓰레드 풀은 대게 필요한 때에만 커질 것이다.

하지만 코루틴은 저렴하다. 그래서 우리는 그들을 간단히 최대 허용 수만큼 시작할 수 있다.

우리가 어떻게 크롤러를 종료시킬 것인지를 아는 것은 중요하다.  
join기능이 해결될 때, worker tasks들은, 살아있지만 종료된 상태 일 것이다.:  
그들은 더 많은 URL들을 기다리고 있지만 오지 않은 것이다.

따라서 메인 코루틴은 그들을 종료시키기 전에, 취소 시킬것이다.  
그렇지 않으면, 파이썬 인터프리터가 종료되고,  
모든 PyObject의 `destructors`를 호출할 때, 살아있는 tasks들은 외칠것이다.

`ERROR:asyncio:Task was destroyed but it is pending!`

그러면 어떻게 `cancel`은 동작하는가?:  
제너레이터들은 당신이 모르는 기능을 가지고 있다.

```python
gen = gen_fn()
gen.send(None)
# 1
gen.throw(Exception("error"))
"""
Traceback (most recent call last):
    ...
Exception: error
```

제너레이터들은 `throw`를 통해서 재진입 되지만, 예외를 발생하고 있다.  
만약 제너레이터의 콜스택의 코드에서 그것을 감지하지 못한다면,  
exception은 불어나서, 맨 위로 올라올 것이다.  
그래서 task의 코루틴을 취소하기 위해서

```python
"""
>>> from asycio import Task
>>> worker = Task(self.work())
>>> worker.cancel()
"""
# method of Task class
def cancel(self):
    self.coro.throw(CacelledError)
```

어떤 `yield from`진술들에서 제너레이터가 정지하면, 이것은 재진입하고 예외를 던진다.  
우리는 취소를 task의 `step`메서드를 통해서 핸들링 할 것이다.

```python
# method of Task class
def step(self, future):
    try:
        next_future = self.coro.send(future.result)
    except CancelledError:
        self.cancelled = True
        return
    except StopIteration:
        return

    next_future.add_done_callback(self.step)
```

이제 task는 그것이 최소 되었는지 알 수 있다. 그래서 이것이 소멸될 때, 소리지르지 않을 것이다.  
`crawl`이 한번 워커를 중지 시키면, 그것은 종료한다.  
이벤트루프는 그것을 보면 코루틴이 종료되었다고 코루틴이 프로그램이 종료되기를 기다린다고 본다.

`loop.run_until_complete(crawler.crawl())`

`crwal`메서드는 모든 메인 코루틴이 해야하는 것을 해낸다.  
이것이 queue에서 URL을 가져오고, 그들을 fetch하고, 새로운 링크를 파싱하는 워커 코루틴이다.

각 워커들은 `work`코루틴을 개별적으로 실행한다.

```python
class Crwaler:
    @coroutine
    def work(self):
        while True:
            url, max_redirect = yield from self.q.get()

            # download page and add new links to self.q
            yield from self.fetch(url, max_redirect)
            self.q.task_done()
```

파이썬은 이 코드가 `yield from`진술을 포함하는 것을 보고, 제너레이터 함수로 컴파일 한다.  
따라서 `crawl`안에, 메인 코루틴이 `self.work`를 10번 호출할 때,  
이것은 실제로 이 함수를 호출 하지 않는다.

그것은 오직 이 코드블록을 바라보는 10개의 제너레이터 객체를 생성할 것이다.

이것은 개별 Task안에 래핑되고, Task는 개별적인 future 제너레이터가 yield하는 것을 받는다.  
그리고 **future가 resolve될 때,** 제너레이터를 `send`에 개별적인 future의 result를 매개변수로 콜하면서 진행할 것이다.  
왜냐하면 제너레이터는 개별적인 스텍프레임을 가지고 있고,  
개별적으로 동작하고, 개별적인 지역변수와 인스트럭션 포인터를 가졌기 때문이다.

워커는 queue를 통해서 그의 친구들과 정렬된다. 그것은 아래처럼 새로운 URL을 기다릴 것이다.

`url, max_redirect = yield from self.q.get()`

queue의 get메서드는 그 자체로 코루틴이다.: 그것은 누군가 queue에 item을 넣을때 까지 정지하고,  
이후 돌아와서 들어온 아이템을 리턴한다.

우연히 아곳은 워커가 crwal의 끝부분에서 메인 코루틴이 그것을 중지할때 멈추는 부분이다.  
코루틴의 관점에서 보았을 때, 이 루프의 마지막 실행은 `yield from` 이 `CancelledError`을 raise할때 일어난다.

워커가 페이지에서 링크들을 파싱하고, 새로운 링크를 큐에 집어 넣을 때, `task_done`을 통해 counter를 줄인다.  
결국 워커는 워커가 모든 URL들이 fetched된 페이지를 fetch하면, 큐에는 더이상 할 일이 남아있지 않다.  
그러므로 이 워커가 `task_done`을 호출하는 것은 카운터를 0으로 줄인다.  
그러면 queue의 `join`메서드를 기다리고 있는 `crawl`은 다시 재개되고, 마무리한다.

`aiohttp` 패키지는 기본적으로 redirects를 따르고, 우리에게 최종 response를 가져다 준다.  
우리가 구분할 수 있건없건, redirects를 크롤러 내에서 핸들링 하고,  
결국 이것은 모든 한 목적지로 가는 redirect path들을 coalesce할 수 있다.:  

* COALESCE: SQL함수로, NULL이 아닌 첫 값을 반환하는 기능  
  만약 해당 URL을 이미 보았다면, 이것은 `self.seen_urls`에 있을 것이고,  
  우리는 이미 이 path에 대해새 다른 엔트리 포인트에서 시도해본적 있는 것이다.

| first entry point | redirects | destination |
| :---------------: | :-------: | :---------: |
|       /foo        |   /baz    |    /quux    |
|       /bar        |   /baz    |    /quux    |

1. /foo에 접근한 워커가 baz를 리디렉트하고, quux로 진행하였다면, baz를 `seen_urls`로 이동시킨다.
2. /bar에 접근한 워커가, baz로 리디렉트 한다는 것을 알면, Fetcher은 baz를 이미 `seen_urls`에 있는 것으로 감안하여 enqueue하지 않는다.
3. 만약 응답이 리디렉션이 아니라 페이지문서였다면, `fetch`는 이것의 링크를 파싱하고, 새로울 것을 enqueue한다.

> STATUS CODE 301 Moved Permanently
>
> 301의 HTTP응답은 location헤더가 포함되는 것이 일반적인데,  
> location헤더에 해당 엔드포인트의 새로운 주소가 포함되어 나온다.  
> 클라이언트는 location헤더의 엔트포인트의 새로운 주소에 해당 요청을 다시 보내게 된다.

```bash
HTTP/1.1 301 Moved Permanently
Location: http://www.example.org/index.asp
```

```python
class Crawler:
    @coroutine
    def fetch(self, url, max_redirect):
        response = yield from self.session.get(
            url, allow_redirects = False
        )
        try:
            if is_redirect(response):
                if max_redirect > 0:
                    # response가 최초 리디렉션 301 MOVED PERMANENTLY 일때,
                    next_url = response.headers["location"]
                    # header에 일반적으로 포함되는 location헤더.
                    if next_url in self.seen_urls:
                        return
                    self.q.put_nowait((next_url, max_redirect - 1))
            else:
                links = yield from self.parse_links(response)
                for link in links.difference(self.seen_urls):
                    self.q.put_nowait((link, self.max_redirect))
                self.seen_urls.update(links)
        finally:
            yield from response.release()
```

만약 이게 멀티스레딩 코드 였다면, race컨디션이 넘쳐났을 것이다.  
예를 들어, 워커가 fetch한 링크가 seen_urls에 있는지 확인하려 한다면,  
그리고 그 워커가 그것을 queue에 넣고 seen_urls로 더하지 않는다면.

만약 이것이 다른 두 operation에 사이에서 interrupt되었다면, 다른 워커들은,  
같은 링크를 다른페이지에서 파싱할 것이고, 또한 그것이 `seen_urls`에 존재하지 않는다고 볼 것이다.  
그리고 그것을 Queue에 더할 것이다.  
그러면 동일한 링크가 Queue에 두번 있는 것이고, 중복된 일과 잘못된 통계를 수집할 것이다.

그러나 코루틴은 오직 `yield from` 인터럽션에만 취약하다.  
이것이 코루틴 코드를 races에 멀티스레딩 코드보다 덜 발생하도록 만드는 주요 차이점이다.:  
멀티스레딩 코드는 lock을 취득하는 것을 통해서 명시적으로 공유자원에 접근해야한다. 그렇지 않으면 방해 받을 수 있다.  
파이썬 코루틴은 기본적으로 방해 받지 않으며, 명시적으로 `yield` 할때만, 컨트롤을 양보한다.

우리는 우리가 Callback기반 프로그램에서 그랬던 것처럼 fetcher클래스가 필요하지 않다.  
그 클래스는 콜백이 결여된 것이었다.:  
그들의 지역변수가 call들 사이에서 유지되지 않았기 때문에, 그들은 I\O를 기다리는 동안 상태를 저장할 장소가 필요하다.

그러나 `fetch`코루틴은 그것의 상태를 일반적인 서브루틴처럼 저장할 수 있기 때문에, class는 더이상 필요하지 않다.

`fetch`가 `work`콜러에게 response를 돌려주는 마무리를 할 때,  
`work`메서드는 `task_done`을 queue에서 호출하여, 다음 URL을 queue에서 가져온다.

`fetch`가 새로운 링크를 큐에 넣을 때, 완료되지 않은 tasks의 count를 증가시키고, `q.join`을 기다리고 있는 메인 코루틴을 정지된 상태로 유지 시킨다.  
그러나, 거기에 unseen링크가 없고, 그것이 queue에서 마지막 URL이었을 때,  
`work`는 `task_done`을 호출하고, 완료되지 않은 tasks의 count는 0으로 떨어진다.  
이벤트는 `join`을 재개시키고, 메인 코루틴은 종료된다.

워커들과 메인 코루틴은 정렬하는 Queue코드는 아래와 같다.

```python
class Queue:
    def __init__(self):
        self._join_future = Future()
        self._unfinished_tasks = 0
    def put_nowait(self, item):
        self._unfinished_tasks = 0
    def task_done(self):
        self._unfinished_tasks -= 1
        if !(self._unfinished_tasks):
            self._join_future.set_result(None)

    @coroutine
    def join(self):
        if self._unfinished_tasks:
            yield from self._join_future
```

메인 코루틴, crawl은 `join`에서 `yield from`된다.  
따라서 마지막 워커가 unfinished_tasks를 0으로 처리하면, 예약한 시그널이 발생해서 crawl로 재진입하고, 종료한다.

`loop.run_until_complete(self.crawler.crawl())`

어떻게 프로그램이 종료 되는가? `crawl`이 제너레이터 함수 이기 때문에, 이것을 호출하는 것은 제너레이터를 리턴한다.  
제너레이터를 진행시키기 위해서, asyncio는 이것을 task안에 래핑한다.

```python
class StopError(BaseException):
    pass

def stop_callback(future):
    raise StopError

class EventLoop:
    def run_until_complete(self, coro):
        task = Task(coro)
        task.add_done_callback(stop_callback)
        try:
            self.run_forever()
        except StopError:
            pass
```

Task가 종료될 떄, 그것을 StopError을 raise하고, 루프는 그것을 시그널로 간주한다.

그러면 이것은 무엇인가? Task는 `add_done_callback`과 `result`라 불리는 메서드가 있다?  
당신은 task가 future를 닮았다고 생각할 수 있다. 당신의 직감을 옳다.  
우리는 당신에게 숨긴 Task class에 대한 세부 디테일을 가져와야 한다. task는 future이다.

```python
class Task(Future):
    """A coroutine wrapped in a Future."""
```

일반적으로 future은 누군가가 set_result를 할때 해제된다.  
그러나 task는 그것의 코루틴이 멈출 때, 자신을 해제시킨다.  
generator가 값을 리턴할 때, 그것을 특별한 `StopIteration`예외를 발생시켰던 것을 기억하라.

```python
class Task(Future):
    def __init__(self, coro):
        self.coro = coro
        f = Future()
        f.set_result(None)
        self.step(f)

    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except CancelledError:
            self.cancelled =True
            return
        except StopIteration as exc:
            self.set_result(exc.value)
            return

        next_future.add_done_callback(self.step)
```

따라서 이벤트 루프가 `task.add_done_callback(stop_callback)`을 호출할 때,  
콜백은 `StopError`를 루프내에서 발생시킨다.  
루프는 멈추고 콜 스택은, `run_util_complete`로 복귀(Unwound to)한다.  
우리 프로그램은 종료 되었다.

## 결론

꽤 자주, 현대적인 프로그램은 CPU-bound하기 보다 I/O-bound한 일이 잦다.  
그러한 프로그램에서 파이썬의 쓰레드들은, 두 곳에서 전부 최악의 경우이다.:  
GIL에 의해서 락을 가진 쓰레드만을 인터프리터가 실행하기 때문에, 병행-컴퓨팅을 방지하고,  
선점형 스위칭은 레이스컨디션을 매우 줄여버린다.  
Async는 가끔 정답인 패턴이다. 하지만 콜백기반 비동기 코드가 커질수록, 이것은 종잡을 수 없이 난잡해지는 경항이 있다.  
코루틴들은 쌍당한 대처법이라 할 수 있다.  
그들은 자신들을 제대로 된 에러 핸들링과, 스택트레이싱을 통해서, 서브코루틴으로 자연스럽게 구성해낸다.

우리가 `yield from`문을 슬쩍 애매하게 본다면, 코루틴은 마치 기존의 blocking-io인 쓰레드가 하는 것처럼 보인다.
우리는 멀티쓰레딩-프로그래밍처럼 코루틴들을 정렬해서, 고전적인 패턴처럼 보이게 할 수도 있다.  
그러므로, 콜백들에 비교해서 코루틴은 멀티쓰레딩으로 익숙한 코더들에게 익숙한 관용구이다.

그러나 우리가, `yield from`문을 제대로 인식하려 할 때,  
우리는 코루틴이 실행 컨텍스트를 넘겨주는 시기 같이, 유심히 체크해야할 포인트 들이 있다.  
쓰레드와 달리 코루틴들은 우리의 코드를 interrupted될 수 있는 것으로 보여준다.  
Glyph Lefkowitz가 적었듯이,
> "쓰레드들은 지역적인 추리를 어렵게 한다. 그리고 지역적인 이해는 소프트웨어 개발에서 가장 중요한 것중 하나이다."

명시적으로 `yielding`하는 것은 반면에 모든 시스템을 시험하는 것이 아닌 루틴으로써,  
루틴을 시험하는 것으로 루틴의 행동을 이해할 수 있도록 한다.

(중략...간편하게 사용할 수 있는 인터페이스가 버전업이 되면서 제공되었다 라는 등의 이야기, `async def`, `await`)

이런 발전들에 불구하고, 기존의 코어 개념은 그대로 있다. 파이썬의 새로운 내장 코루틴들은,  
통상적으로, 제너레이터와 구분되지만, 매우 유사하게 동작한다.  
게다가, 그들은 파이썬 인터프리터 내에서, 그 실행부를 공유한다.  
`Task`, `Future`, `event loop`같은 것들은 `asyncio`에서 계속 그들의 역할을 맞을 것이다.

이제 당신은 어떻게 `asyncio` 코루틴이 동작하는지 알았으니, 세부적인 것은 조금 잊어도 좋다.  
기계공은 매끈한 인터페이스 뒤에 dlTek.  
그러나 당신이 구조를 이해한다면, 너의 코드를 요즘 비동기 환경에서 더욱 효과적이고 올바르게 강화해줄 것이다.

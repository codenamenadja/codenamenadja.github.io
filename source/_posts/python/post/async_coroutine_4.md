---
title: 비동기 코루틴 번역-4
p: python/post/async_coroutine_4
date: 2019-06-19 18:11:02
tags: ['python', 'python post']
---


제너레이터는 생성된 제너레이터 객체가 그 자체로, 개별적인 스텍프레임을 보유한다.
파이썬은 내부적으로C-stackFrame을 Extends하여,
| PyGenObject | PyFrameObject |      PyCodeObject |
| :---------- | ------------: | ----------------: |
| gi_frame    |       f_lasti | gen_fn's bytecode |
| gi_code     |      f_locals |                   |

이런 구조를 띈다고 할 수 있는데, 제너레이터가 내장한 스텍프레임은 프레임 오브젝트를 생성하고 해당 프레임은 프레임 객체로서 f_lasti인 마지막 인스트럭션을 제너레이터 내부에 저장한다.

제너레이터 코드 자체인 gi_code는 PyCodeObject를 생성하는데, 내부 제너레이터함수의 바이트 코드 안에 모든 정의를 담고 있다.

결론적으로 지향한 C의 프레임으로 프레임오브젝트를 생성하고 그것이 CodeObject에 캡슐화 되어 보관된다.

```python
class Future:
	def __init__(self):
		self.result = None
		self._callbacks = []
	
	def add_done_callback(self,fn):
		self._callbacks.append(fn)

	def set_result(self,result):
		self.result = result
		for f in self._callbacks:
			fn(self)
```
Future은 OS의 IO에 바운드 되는, 생성된 socket들에게 걸어놓은 이벤트들에 대해서 OS가 포트를 통해서 특정 리스폰스를 받았을때,
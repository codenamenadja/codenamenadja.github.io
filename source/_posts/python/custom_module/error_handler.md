---
title: 에러처리 Decorator과 logging, write to csv
p: python/custom_module/error_handler
date: 2019-08-09 17:22:43
tags: ['python', 'custom module']
---


## Error hanlder with logging

### [github repo](https://github.com/codenamenadja/python_cutsom-error_handler)

### 기초 사용법

- Expected Error

> 2번째 요소를 지정하면서 특정상황에 일으키는 예견된 에러

```python
raise ValueError("some message", "info")
raise PermissionError("some message", "warning")
raise FileNotFoundError("some message", "errror")
raise AttributeError("some message", "debug")
raise ConnectionRefusedError("some message", "critical")
raise ConnectionAbortedError("some message", "fatal")
```

   1. logs into curdir filename debug_expected.log
   1. writes as, `asctime - name - levelname - {message}`
      1. {message}: `filename - lineno - message`

- UnExpected Error

> 2번째 요소가 지정 되지 않는 실제 에러

```python
raise ValueError("message")
```

   1. logs into curdir filename debug_unexpected.log
   2. writes same as upper,
   3. run logger.log_to_csv()

```python
    def log_to_csv(logger_name: Optional[str] = "unexpected") -> bool
        """
        read {logger_name}_debug.log file
        write into csv as thread reads line.

        if file exists, write over ends of the line
        """
```

## custominzing to do

- 예상 외 에러를 발생하면 CSV파일이 모두 씌여지고, 해당 CSV파일을 S3저장소에 업로드
- 마지막줄 기준으로 날짜가 달라졌다면, 오늘 이전 로그를 이메일로 전송(그렇다면 1일 최대 1회 이메일로 파일전송)

## more to do

- 순수한 에러핸들러 데커레이터와 로깅이 연동된 데커레이터를 분리
---
title: http_1.0_sementics
p: http/2_http_sementics
date: 2019-09-17 23:09:02
tags: ["http"]
---


## index

1. [**Simple Form Trasportation: 간단한 폼 전송**][i1]
2. [**File trasnport with form: 폼을 통한 파일 전송**][i2]

## Simple_form_transportation

[i1]: #simple_form_transportation

```bash
# client
curl --http1.0 -d title="art of..." -d author="jono" http://127.0.0.1:18888/

# server
Content-Length: 27
Content-Type: application/x-www-form-urlencoded
User-Agent: curl/7.58.0

title=art of...&author=jono

# httpclient
http http://127.0.0.1:18888/ username=sample password=asdqrfrg -f

# server
Content-Length: 33
Content-Type: application/x-www-form-urlencoded; charset=utf-8
User-Agent: HTTPie/0.9.8

username=sample&password=asdqrfrg
```

위 커맨드로 처리하는 바디는 브라우저의 웹폼과는 달리 지정된 문자열의 특수문자를 변환없이 그대로 연결한다.  
`RFC 1866`에서 책정한 변환 포멧에 따라,

* 알파벳
* 숫자
* 별표
* 하이픈
* 마침표
* 언더스코어

의 종류 외 문자는 변환이 되어 아래와 같은 형식이 된다.

`title=Head First PHP & MySQL&authre = Lynn Beighley, Micheal Morrison`  
`title=Head+First+PHP+%26+MySQL&author=Lynn+Beighley%2C+Micheal+Morrison`

curl커맨드에서는 `--data-urlencode`옵션을 통해서 `RFC 3986`에 정의된 방법으로 변환한다.  
이 경우에는 `RFC 1866`과 달리 공백이 `+`가 아닌 `%20`으로 처리한다.

____

## File_transport_WITH_form

[i2]: #file_transport_with_form

HTML Form에서 `multipart`라는 폼 형식 인코딩 타입을 통해서, 파일을 전송할 수 있다.  
이는  `RFC 1867`에 정의된다.

```html
<form action="POST" enctype="mulitpart/form-data">
</form>
```

HTTP응답은 한번에 한 바디만 반환하므로, 빈줄로부터 `Content-Length`바이트 만큼 읽어내면 종료된다.  
하지만 멀티파트를 통하는 경우 한번의 요청으로 복수의 파일의 경계를 받는쪽에서 처리해야 한다.

`Content-Type`은 모두 `multipart/form-data`이지만, 또 하나의 속성인 경계 문자열이 있다. 각 따라 독자적인 포멧으로 랜덤하게 생성한다.

```bash
Content-Type: multipart/form-data; boundary=---WebKitFormBoundaryy0YfbccgoID172j7

------WebKitFormBoundaryy0YfbccgoID172j7
Content-Disposition: form-data; name="title"

The Art of Community
------WebKitFormBoundaryy0YfbccgoID172j7
Content-Disposition: form-data; name="author"

Jono Bacon
------WebKitFormBoundaryy0YfbccgoID172j7
```

```bash
POST / HTTP/1.1
Host: 127.0.0.1:18888
Accept: */*
Accept-Encoding: gzip, deflate
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 2719
Content-Type: multipart/form-data; boundary=--------------------------925161045727238542287839
Postman-Token: cd5fc7b3-3090-4cc7-921e-223c3defc0a5
User-Agent: PostmanRuntime/7.6.0

----------------------------925161045727238542287839
Content-Disposition: form-data; name=""; filename="3_8_buffer_io.c"
Content-Type: text/x-c

#include <stdio.h>

int main(void)
{
    FILE *in, *out;
    struct pirate
    {
        char name[100];
        unsigned long booty;
        unsigned int beard_len;
    };
    struct pirate p;
    struct pirate blackbeard = {"Edward Teach", 950, 48};
}
----------------------------925161045727238542287839
Content-Disposition: form-data; name=""; filename="sample.txt"
Content-Type: application/javascript

sample file_2 content-data
----------------------------925161045727238542287839--
```

curl에서는 `-d` 대신 `-F`를 사용하는 것만으로 enctype="multipart/form-data"로 설정한다.

`curl --http1.0 -F "attachment-file@test.txt;filename=sample.txt;type=text/html" http://127.0.0.1:18888/`

* filename과 type은 메뉴얼하게 지정하며, 내용은 text.txt에서 취득한다.

## Content-Negotiation

[i3]: #content-negotiation

* 컨텐츠 협상에 사용되는 대상과 헤더

| Header          |           Response            | Negotiation-Target |
| :-------------- | :---------------------------: | :----------------: |
| Accept          |        Content-Type 헤더        |      MIME 타입       |
| Accept-Language | Content-Language 헤더 / html 태그 |       표시 언어        |
| Accept-Charset  |        Content-Type 헤더        |      문자의 문자셋       |
| Accept-Encoding |      Content-Encoding 헤더      |       바디 압축        |

### 파일_종류_결정

`Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8`

* `image/webp`
* `*/*;q=0.8`

q는 우선순위르 나타내는 품질 계수로 0~1사이 수치로 표현되며, 기본값은 1.0으로 이때는 생략된다.  
웹서버가 `Webp`("구글이 권장하는 PNG보다 20%절약 포멧")를 지원하면 webp를,  
그렇지 않으면 PNG등 다른 포멧(q=0.8)을 서버에 보낼 것을 요구한다.

서버는 Request에서 요구한 형식중에서 파일을 반환.  
우선 순위를 해석해, 위에서부터 차례로 지원하는 포멧을 찾고, 그 포멧을 반환한다.  
만약 일치하는 형식이 없으면 서버가 `406 NOT Acceptable`을 반환한다.

### 표시_언어_결정

`Accept-Language: en-US,en;q=0.8,ko;q=0.6`

1. en-US
2. en
3. ko

순의 우선순위로 요청 헤더를 보낸다.  

### 문자셋_결정

`Accept-Charset: windows-949,utf-8;q=0.7,*;q=0.3`

현재는 어떤 브라우저도 `Accept-Charset`헤더를 송신하지 않는다.  
대부분 브라우저가 문자셋 인코더를 내장하고 있어, 대부분의 문자를 처리할 수 있기 때문이다.  
문자셋은 MIME타입과 세트로 Content-Type헤더에 실려서 전달된다.

`Content-Type: text/html; charset=UTF-8`

HTML의 경우 문서 안에 쓰기도 한다. `RFC 1866`의 HTML/2.0으로 이용할 수 있다.  
HTML을 로컬에 저장했다가 다시 표시하는 경우도 많으므로, 이용된다.

```html
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
```

http-equiv태그는 HTTP헤더와 똑같은 지시를 문서 내부에 삽입해서 반환하는 상자.  
HTML5의 경우, 아래와 같이 표기 가능하다.

```html
<meta charset="UTF-8">
```

---
title: "http/1.0의 신택스: 기본이 되는 네가지 요소"
p: http/1_http_1.0_basic_four_feature
date: 2019-09-06 20:03:56
tags: ["http"]
---

## index

1. [**HTTP 역사 및 대표 기관명칭**][i1]
2. [**HTTP 0.9**][i2]
3. [**HTTP 1.0 전환**][i3]
4. [**Content-Type의 값 MIME타입**][i4]
5. [**메서드와 스테이터스 코드**][i5]
6. [**URL**][i6]
7. [**BODY**][i7]

## HTTP의_역사와_담당기관

[i1]: #http역사와_담당기관

* 1990년: HTTP/0.9, CREN에서 근무하던 팀 버너스리가 최초 웹서버 CERN HTTPd 개발
* 1996년: HTTP/1.0
* 1997년: HTTP/1.1
* 2005년: HTTP/2

|   이름   |                       정식 명칭                        |                       역할/의미                        |
| :----: | :------------------------------------------------: | :------------------------------------------------: |
|  IETF  |          Internet Engineering Task Force           | 인터넷의 상호 접속성을 향상시키는 것을 목적으로 만들어진 임의 단체 (통신프로토콜 관리자) |
|  RFC   |                Request For Comments                |    IETF가 만든 규약문서         (상호 접속성 유지를 위한 사양서 모음)    |
|  IANA  |        Internet Assigend Numbers Authority         |         포트번호와 컨텐츠타입등 웹에 관한 데이터베이스를 관리하는 단체         |
|  W3C   |             World Wide Web Consortium              |      웹 관련 표준화를 하는 비영리 단체    (브라우저에 특화된 기능 책정)      |
| WHATWG | Web Hypertext Application Technology Working Group |         웹 관련 규격을 논의하는 단체, W3C와 겸하는 멤버가 많다.         |
****

## HTTP_0.9

[i2]: #http_0.9

* go 에코서버

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "net/http/httputil"
)

func handler(writer http.ResponseWriter, req *http.Request) {
    dump, err := httputil.DumpRequest(req, true)
    /*
    func dumprequst(req *http.request, body bool)([]byte, error)
    */
    if err != nil {
        http.Error(writer, fmt.Sprint(err), http.StatusInternalServerError)
    return
    }
    fmt.Println(string(dump))
    fmt.Fprintf(writer, "<html><body>hello</body></html>")
}

func main() { // 초기실행부
    var httpServer http.Server
    http.HandleFunc("/", handler)
    // "/"에 접속이 있을 때, handler함수를 호출
    log.Println("start http listening :18888")
    httpServer.Addr = ":18888"
    // 18888포트로 설정
    log.Println(httpServer.ListenAndServe())
}
```

* request 전달

```bash
# client
$ curl --http1.0 http://localhost:18888/greeting
# response
<html><body>hello</body></html>

# server: request
GET / HTTP/1.0
Host: 127.0.0.1:18888
Connection: close
Accept: */*
User-Agent: curl/7.58.0
```

0.9버전은 1.0과 호환되지 않기 때문에, 0.9요청을 전달하는 것은 현재는 불가능하다.  
0.9버전 당시에 content/type은 text/html밖에 존재하지 않음  

* 검색 기능

```bash
# client
$ curl --http1.0 --get --data-urlencode "search word" http://
localhost:18888/greeting

# server
GET /?search%20word HTTP/1.0
Host: 127.0.0.1:18888
Connection: close
Accept: */*
User-Agent: curl/7.58.0
```

****

## http_0.9에서_1.0으로

[i3]: http_1.0

* HTTP/0.9의 프로토콜로는 할 수 없는 일
  * 하나의 문서를 전송하는 기능만 존재(`<html> -> </html>`)
  * 컨텐츠 타입은 모두 text/html로 가정. 다운로드할 컨텐츠를 서버가 바꿀 수 없었다.
  * 클라이언트에서 검색이외 요청을 보낼 수 없었다.
  * 새로운 문장을 전송하거나 갱신 또는 삭제 불가?(소켓유지?)
  * 요청이 올바른지 혹은 서버가 올바르게 처리했는지 전달할 수 없음.

```bash
curl --get -v --http1.0 http://127.0.0.1:18888/

*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 18888 (#0)
# >로 시작하는 행이 클라이언트에서 서버로 갈 내용
> GET / HTTP/1.0
> Host: 127.0.0.1:18888
> User-Agent: curl/7.58.0
> Accept: */*
> 
# 여기까지
* HTTP 1.0, assume close after body
# <로 시작하는 행은 서버 응답
< HTTP/1.0 200 OK
< Date: Fri, 06 Sep 2019 11:53:03 GMT
< Content-Length: 31
< Content-Type: text/html; charset=utf-8
< 
* Closing connection 0
<html><body>hello</body></html>
```

* 1.0으로 버전업 변경점
  * 요청 시 메서드가 추가
  * 요청 시 HTTP버정이 추가(HTTP/1.0)
  * 헤더가 추가(Host, User-Agent, Accept)
  * 응답 시 HTTP버전과 스테이터스 코드가 포함
  * 요청과 같은 형식의 헤더가 포함됨
  
ARPAnet에서 부터 시작된 전자메일(RFC822)이 HTTP보다 더욱 발달 했었기 때문에, 많은 참조, 승계가 일어남.

1. Req header From Client
   * User-Agent: 클라이언트가 자신의 어플리케이션 이름을 전달 하는 키값.
   * Referer: 서버에서 참고할 수 있는 추가 정보, 요청을 보내는 시점의 URL등을 포함. 보안때문에, 사양이 당초 보다 크게 변경됨.
   * Authorization: 특별한 클라이언트에만 통신을 허가할 떄 인증 정보를 서버에게 전달

2. Res header From Server
   * Content-Type: 파일 종류. MIME타입 이라는 식별자를 기술.(전자메일을 위해 만들어짐)
   * Content-Length: 바디 크기. 압축이 이루어지는 경우 압축 후의 크기
   * Content-Encoding: 압축이 이루어진 경우 압축 형식
   * Date: 문서 날짜

```bash
# 윈도우 7의 익스플로러 10 버전으로 User-Agent설정
curl -v --http1.0 -A "Mozilla/5.0(compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)" http://127.0.0.1:18888/

# server output

GET / HTTP/1.0
Host: 127.0.0.1:18888
Connection: close
Accept: */*
User-Agent: Mozilla/5.0(compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)

# client output

*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 18888 (#0)
> GET / HTTP/1.0
> Host: 127.0.0.1:18888
> User-Agent: Mozilla/5.0(compatible; MSIE 10.0; Windows NT 6.1; Trident/6.0)
> Accept: */*
> 
* HTTP 1.0, assume close after body
< HTTP/1.0 200 OK
< Date: Fri, 13 Sep 2019 07:41:43 GMT
< Content-Length: 31
< Content-Type: text/html; charset=utf-8 # MIME타입 문자열
< 
* Closing connection 0

```

****

## MIME타입

[i4]: #mime타입

### MIME타입_개요 (RFC 1341)

RFC1341에 처음 등장하여, 전자메일을 위해 만들어졌으며, 파일의 종류를 구별하는 문자열이다.  
인터넷이 보급된 시기는 MS-DOS, 윈도우3.x, 맥 OS7등이 사용되던 시기.

OS별 파일의 구별

* 윈도우: 파일 확장자를 통해 구별
* 맥: Resource fork라 불리는 메타정보로 파일 종류를 판단

지금도 위 사항은 기본적으로 유지되고 있음.

### Content-Type의 등장 (RFC 1049)

이 시기(1988년)에는 Content-Type인 MIME_TYPE(아직 MIME타입이 아니라 값)에는
1. POSTSCRIPT
2. TEX

와 같은 것들이 있었다.

**대항목/상세**, 같은 식의 표기방식을 채택한 1992년 RFC 1341에서(HTML 미등장)

1. 'text/plain'
2. 'image/jpeg'
3. 'multipart/form-data'

같은 포멧을 위한 표기 방법이 등장하게 되면서 MIME타입이라 공식화 하게된다.

1. 서버에서 클라이언트로 데이터를 전송할 때, 컨텐트 타입을 헤더에 MIME타입값으로 정의하여 send하면,  
2. 브라우저 혹은 무엇이든 요청을 보낸 클라이언트 어플리케이션은 그것을 수용할 수 있는지 없는지 판단하고
3. 서버와 소통을 통해 최종 결정하여 body를 받는다. 이 과정을 Negotiation이라 한다.

RFC 1590에서 새로운 종류의 MIME타입을 IANA에 등록신청하는 절차를 구성하였다.  
RFC 3023에서, JSON, XML등의 MIME타입이 공식화 되었다.  
이로써, XML을 기반으로 한 SVG이미지를 application/xml이 아니라,  
Image/svg+xml로 표기 할 수 있게 되었다.

### Content-Type과_보안

특정 타입으로 BODY를 해석해 달라는 요청을 Content-Type대신 확장자를 사용하려고 해도, 클라이언트에 따라 전송되지 않을 수 있다.  

2000년 초기까지는 CGI를 이용한 접속 카운터(숫자가 들어간 이미지를 생성하는 Pearl등의 스크립트 언어 프로그램)로 대체.  
이 카운터는 .cgi라는 확장자로 HTML에서 아래와 같은 식으로 CGI를 호출하여 사용하였다.

```html
<img src="cgi-bin/counter.cgi">
```

이 경우, cgi프로그램이 `Content-Type:image/gif`같은 헤더를 생성하여 처리.

그러나 익스플로러는 인터넷 옵션에 따라, MIME타입이 아니라 내용을 보고 파일 형식을 추측하기도 한다.  
이런 동작을 *content sniffing*이라 한다.  
서버에서 잘못된 처리로 타입과 언매치한 바디를 주었을 때, 해결해 줄 수 있는 장점이 있으나,  
브라우저가 추측하는 정보가 잘못되어, 예상 외의 분석, 실행을 하는 경우에 보안의 문제가 일어난다.  
예를 들어 자바스크립트가 아닌데, 자바스크립트로 해석하면, 엄격한 규칙에 따라 그 응답을 브라우저에서 실행하지 않는 경우등이 있다.

서버에서 다음과 같은 헤더를 통하여, 브라우저의 추측을 금지하도록 지시하는 방식을 주로 사용하고 있다.

`X-Content-Type-Options: nosniff`
****

## 메서드와_STATUS_CODE

[i5]: 메서드와_status_code

### HTTP의_또다른_메타_뉴스그룹

인터넷 이전의 주요 미디어로서, 지금은 사용되지 않는 뉴스그룹은 분산 아키텍쳐로 구성되어 있다.  
사용자는 서버에 구독하는 최신기사를 요청하고 기사가 있으면 가져온다.  
웹처럼 모든 사용자가 한 곳의 서버에 접속하러 가는 것이 아니라, 복수의 서버가 master/slave구조로 연결.

슬레이브의 서버가 마스터로 접속하고 정보를 가져와 로컬에 저장.  
스토리지 용량제한으로 오래된 기사는 삭제한다.  
마스터서버를 참조하는 복수의 슬레이브 서버, 그리고 슬레이브에 접근하는 클라이언트 간 통신에 사용된 것이,

`NNTP(Network News Transfer Protocol)`이다. `(RFC 977)`

HTTP0.9보다 5년 빠른 1986년에 정식화 되었으며, 메세지 포멧은 그보다 3년 전인 `RFC 850`에서 등장하였다.  
이 포멧도 전자메일을 영향을 받아 정식화되었고, 마찬가지로 헤다와 본문이 있고, 사이에 빈줄이 들어가는 구성이다.  
HTTP는 뉴스그룹 프로토콜로 부터 `METHOD`, `STATUS CODE`를 이어받는다.

### METHOD

지정된 주소에 있는 리소스에 대한 조작을 지시한다.

* 뉴스그룹 메서드
  1. LIST: 그룹 목록 취득
  2. HEAD: 헤더 취득
  3. BODY: 기사 취득
  4. POST: 투고

HTTP의 경우는 파일 시스템과 같은 설계철학으로 만들어졌다.
기본으로 `GET`, `HEAD`, `POST`는 가장 많이 쓰이는 메서드이며,  
`PUT`, `DELETE`는 사용빈도에 따라 필수가 아니라 구현에 따라 Optional한 기능이 되었다.

1. GET: 헤더와 컨텐츠 요청
2. HEAD: 헤더만 요청
3. POST: 새로운 문서 투고
4. PUT: 이미 존재하는 URL의 문서를 갱신
5. DELETE: 지정된 URL의 문서를 삭제, 성공시 해당 URL은 무효

`curl --http1.0 --request POST http://127.0.0.1:18888/`

### STATUS_CODE

뉴스그룹 프로토콜에서 가져온 기능으로, 5가지 카테고리로 나눌 수 있다.

* 1XX번대: 처리가 계속됨을 나타낸다.
* 2XX번대: 성공했을 때의 응답. 200 OK 정상종료
* 3XX번대: 서버에서 클라이언트로의 명령. 오류가 아니라 정상처리의 범주이며, `리디렉트`, `캐시 이용`등을 지시한다.
* 4XX번대: 클라이언트가 보낸 요청에 오류가 있다.
* 5XX번대: 서버 내부에서 오류가 발생했다.

### 리디렉트

Redirect를 통해 다른 페이지를 GET하라는 권고로, 해당 요청에서는 시나리오가 없으니, 다시 요청을 날리라는 표현.

|          코드번호          | 메서드 변경 | 영구적/일시적 |                    캐시                    |           설명           |
| :--------------------: | :----: | :-----: | :--------------------------------------: | :--------------------: |
| 301 Moved Permanently  |   O    |   영구적   |                    한다                    | 도메인 전송, 웹사이트 이전, HTTPS |
|       302 Found        |   O    |   일시적   | 지시에 따름(`Cache-Control`, `Expires`헤더를 통해) |   일시적 관리, 모바일 기반 전송    |
|     303 See Other      |   허가   |   영구적   |                   하지않음                   |      로그인 후 페이지 전환      |
| 307 Temporary Redirect |        |   일시적   |                  지시에 따름                  |    `RFC 7231`에서 추가     |
|  308 Moved Permanetly  |        |   영구적   |                    한다                    |    `RFC 7538`에서 추가     |

* 영구적: 영구적인 접근 차단.(무효한 자원, HTTP -> HTTPS)
* 일시적: 이동하려던 페이지에 언젠가는 이동 가능(존재하는 자원)
* 303의 경우: 로그인 같은 경우는 보통 POST메서드로 이루어지는데,  
  자원에 대한 요청이 아니기 때문에, 반환할 컨텐츠가 없으며, 동시에 별도로 처리할 페이지가 있는 경우이다.  
  따라서 해당 리디렉션은 일시적으로 접근 못하는 것이 아니라 정상적인 시나리오 내에서,  
  영구적으로 가야하는 곳을 다시 지정하는 시나리오.
* 301의 경우: 요청된 페이지가 다른 장소로 이동했을떄, 기존도메인에서 301을 처리해주면,  
  **구글검색엔진은 해당 페이지에 대한 평가를 리디렉션으로 상속**시킨다.

Redirection의 경우 대부분 헤더에 `Location`키값이 존재히는데,  
`curl --location --max-redirs 5 GET http://127.0.0.0.1:18888/get_to_redirect/`  
Response-헤더에 location이 존재할 경우, 해당 값으로 GET 최대 5회까지 리디렉션을 보내고, 기존 Request-헤더를 유지한다.
> 구글은 리디렉트의 사이드 이펙트를 염려하며, 권장 3회 이하, 최대 5회 이하라는 가이드 라인을 제시하였다.

`RFC 2616`에서는 302의 코드설명상,
> 메서드변경을 허가가 필요하도록 하였고, 현재 유저에이전트는 대부분 GET으로 변경한다.

`RFC 7231`애서는,
> 302를 301처럼 메서드 변경을 허용하고, 변경허가가 필요한 307, 308을 추가한다.
****

## URL

[i6]: #url
`RFC 1738`에서 등장하였으며, 상대적 URL은 `RFC 1808`에서 등장.  
이들은 모두 HTTP/1.0보다 빠른 시기의 문서로,  
HTTP/0.9중에 HTTP/1.0를 계획하에 먼저 규격화 되었다.

### URL의_구조

일반적인 경우

`스키마://호스트명/경로`의 형태로 구성이 되지만, URL사양에 포함되는 모든 요소가 들어간 경우는  
`스키마://사용자:패스워드@호스트명:포트/경로#프래그먼트?쿼리`로 구성된다.

1. `스키마`: 스키마해석은, 브라우저의 책임이며, https. mailto, ftp등이 올 수 있다.  
  로컬 파일을 브라우저로 열면 `file:`로 표시된다.
2. `사용자:패스워드`: Basic인증 방식이며, 로그에 URL이 남게되어 유출되기 때문에, 웹 시스템에서는 사용하지 않는다.  
3. `호스트명`: DNS서버에, 엔드포인트로는 실제 IP라우팅 주소값을 특정하여 목적지를 찾아간다.
4. `포트`: 스키마에 따라서 기본 well-known-port로 처리되며, 한 서버에 여러포트를 사용하는 경우 복수 서비스를 운영할 수 있다,
5. `프래그먼트`: HTML문서 내의 링크 앵커를 지정
6. `쿼리`: 해당 웹페이지에 대해서 특정 파라미터를 부여하고 싶은 경우 사용

### URL과_국제화

기존엔 URL의 도메인 이름으로 영숫자와 하이픈만 사용 가능.  
2003년 `RFC 3492`에서 IDN(International Domain Name)을 표현하는 인코딩 규칙 **PUNY CODE**가 정해져,  
퓨니코드가 구현된 브라우저에서는 다국어를 도메인 네임으로 사용할 수 있다.  
정해진 규칙에 따라, 영숫자이외의 문자를 반각영숫자로 치환해 요청을 보낸다.

`한글도메인.kr` -> `xn--bj0bj3i97fq8o5lq.kr`이 된다.

퓨니코드는 반드시 `xn--`로 시작되는 문자열을 생성한다.

****

## BODY

[i7]: #body
HTTP1.0이후부터 요청, 응답 모두 헤더를 포함하게 되면서, 헤더와 바디를 분리할 필요가 있어졌다.  
전자메일과 같이 헤더의 끝에 빈줄`\n`을 넣으면 그 이후는 모두 바디가 된다.

그러나 바디의 전송할 때, 데이터를 저장하는 포멧이 2종류가 존재하여서, 용도에 맞게 구분법이 다르다.

1. `Content-Encoding`의 압축알고리즘을 통한 압축전송과 HTMLFORM, XMLHttpRequest를 사용한 Request측 바디 전송
2. 청크 형식의 바디 송신

```bash
헤더1: 값
헤더2: 값
Content-Length: 바디의 바이트 수

--이 줄부터 지정된 바이트 수 만큼 바디--
```

> Head요청시, 헤더만을 요구하는 요청이지만, Content-Length와 E-Tag등을 바르게 전송해야한다.

* Curl커맨드의 바디 획득 옵션

|       옵션| 용도     |
| :---: | ---: |
|-d, --data, --data-ascii|변환 완료된 텍스트데이터|
|--data-urlencode|텍스트 데이터 ascii변환|
|--data-binary|바이너리 데이터|
|-T Filename, -d @Filename|보내고 싶은 데이터를 파일에서 읽어온다.|

송신시 바디를 서버에 보내려면 -d옵션을 사용한다.  
그럴 경우 기본적으로 `Content-Type:application/x-www-form-urlencoded`가 된다.  

* JSON을 전송하고 싶은 경우

```bash
curl -d "{\"hello\":\"world\"}" -H "Content-Type: application/json" http://127.0.0.1:18888/

# JSON파일에서 읽어서 전송
curl -d @test.json -H "Content-Type: application/json" http://127.0.0.1:18888/
```

`application/x-www-from-urlencoded`같은 형식은 http0.9에서 파생되어,  
http1.0보다 먼저 표준화된 HTML 2.0의 사양이다.  
이는 `RFC 1866`에서 표준화 되었으며, RFC로 정의된 html은 이게 처음이자 마지막.  
`RFC 2854`의 결정으로 HTML사양 결정은 `W3C`로 넘어간다.

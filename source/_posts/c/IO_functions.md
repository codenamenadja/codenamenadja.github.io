---
title: 소켓과 스트림을 이해하기 위한 C의 IO함수들
p: c/IO_functions
date: 2019-08-19 18:34:26
tags: ['c']
---


## Index

1. [**개요**][i1]
2. [**입출력 함수**][i2]
3. [**표준 입출력과 버퍼**][i3]
4. [**File과 Stream, 그리고 일반파일 입출력**][i4]

## 개요

[i1]: #개요

콘솔 입출력을 위한 입력 스트림과 출력 스트림은 프로그램이 실행되면 자동으로 생성되고, 프로그램이 종료되면 자동으로 소멸되는 스트림이다.

* 표준 스트림
| name   | desc      | target |
| :----- | :-------- | :----- |
| stdin  | 표준 입력 스트림 | 키보드 대상 |
| stdout | 표준 출력 스트림 | 모니터 대상 |
| stderr | 표준 에러 스트림 | 모니터 대상 |

stdin이 **입출력 리디렉션**을 통해서 stdout의 파일로 전송되고 해당 바이트 만큼 stdin을 fflush하여 stdin은 비워진 상태로 인터럽션을 일으키지 않는다.

* [basic concept of Linux / about file 링크](https://codenamenadja.github.io/2019/08/12/linux/basic_concept/#특수_파일)

하드웨어 장치를 추상화하는 디바이스 파일로서 커널이 파일로서 관리하는 기본적인 2분류에 대해서 설명했었다.

키보드 디바이스 파일은,
1. 캐릭터 디바이스 파일로서 해당 파일에 캐릭터가 들어오면 바로 인터럽션을 일으키고,
2. 영어의 경우 1바이트를 읽어내고 언어 설정에 따라 해당 키보드 입력 값을 UNICODE캐릭터로 변환
3. 모니터 출력 디바이스파일로 전송한다.
4. 브라우저프로세스를 예를 들면, Keyup이라하는 이벤트는, 키보드 인터럽션에 대해서, output스트림을 stdout이 아닌 브라우저 프로세스 메모리에 대해 연결해 놓은 것이다.
5. 프로세스에서 처리가 끝나면, 부분적인 렌더링을 개시한다. 내가 입력한 것이 모니터로 전달한 것이 아니라, 내가 입력한 값이 속성으로 전달되고, 프로세스가 그것을 모니터 출력으로 연결하는 것이다.
6. 그래서 프로세스에서 value를 조정해서 객체의 속성을 브라우저에 인식시키면, 해당 프로세스의 로직이 바로 모니터 out에 전달하게 하는지 아닌지는 로직에 달려있다.
7. stdin은 전달하고 나면 바로 fflush되어 해당 디바이스 파일은 비워지게 된다.

만약 fflush되어 해당 피일이 비워지지 않는다면, 뭔가 에러가 날 가능 성이 있는데,
가장 확실한 에러는 다음에 값을 입력해서 키보드 디바이스파일에 캐릭터가 들어왔다는 인터럽션이 일어났을 때,
읽어야할 바이트가 1바이트인데, 앞에서 부터 1바이트를 읽는다하면, 이번에 입력된 값과 추후 들어올 값들은 모두 누적되기만 하는 것이다.
___

## 입출력_함수

[i2]: #입출력_함수

### 문자 단위 입출력 함수

* 문자 출력 함수

**함수 호출 성공시 쓰여진 문자정보, 실패 시 `EOF` 반환**

1. putchar

    `int putchar(int c);`

    인자로 전달된 문자 정보를 `stdout`으로 전송하는 함수

2. fputc

    `int fputc(int c, FILE *stream);`

    위와 동일하지만, `FILE *stream`의 파일객체를 지정할 수 있다. 출력 스트림이 모니터 디바이스파일이 아닌, 일반 파일로 전송하는 것이 가능하다.

    따라서 `stdout`을 2번째 매개변수로 전달하면 `putchar`과 동일한 함수가 된다.

* 문자 입력 함수

**파일 끝에 도달하거나, 함수호출 실패 시 `EOF` 반환**

1. getchar

    `int getchar(void);`

    모니터 출력 대응용으로 하나의 문자를 받아서 반환한다.

2. fgetc

    `int fgetc(FILE *stream)`

    `fgetc`와 달리 입력 받을 스트림을 지정할 수 있다.

### 예제_1

* 단순 입출력 사용

```c
# include <stdio.h>

int main(void)
{
    int ch1, ch2;

    ch1 = getchar(); // 문자 키 입력
    ch2 = fgetc(stdin); // 엔터 키 입력

    putchar(ch1) // 문자 출력
    fputc(ch2, stdout) // 엔터 키 출력
    return 0;
}
/*
console->
p (p 누르고 엔터)
p (p출력, 엔터\n 으로 ascii 10인 값으로 출력)
(엔터로 인한 줄바꿈)
*/
```

* 문자 입출력에서의 `EOF`

```c
# include <stdio.h>

int main(void)
{
    int ch;

    while(1)
    {
        ch = getchar();
        if(ch==EOF)
            break;
        putchar(ch);
    }
    return 0;
}
/*
console ->
Hi~ (사용자 입력+엔터)
Hi~ ()
I like C lang. (사용자 입력+엔터)
I like C lang.
^Z (EOF를 반환시키는 CTRL+Z , CTRL+D)
*/
```

getchar()함수는 문자 하나를 입력 받기 위한 함수 이지만, 만약 키보드 디바이스파일에 문자 하나이상 입력되었다면,
그 값이 비워질 만큼, getchar()을 누적해서 실행하여 해당 변수에 더한다.
getchar()자체는 하나만을 리턴하는게 맞지만 만약 입력값이 남아있다면 파일이 빌 때 까지 계속 실행하는 것이다.

* 문자 반환형이 int 인 이유는?

반환되는 것은 1바이트 크기의 문자인데, 반환형이 int인 이유는
반환하는 값 중 하나인 `EOF` 는 -1로 정의된 상수이다.

따라서 char형 이라면, 그리고 char을 unsigned char로 처리하는 컴파일러에 의해 컴파일이 되었다면, EOF는 반환의 과정에서

엉뚱하게도 양의 정수로 형변환이 되어버리고 만다.
그래서 어떤 상황에서도 -1을 인식 할 수 있는 int형으로 반환형을 정의해 놓은 것이다.
___

### 문자열 단위 입출력 함수

* 문자열 출력 함수

**성공시 음수가 아닌 값을, 실패 시 `EOF`반환**  

**문자 열의 끝에는 항상 `Null`이 1바이트 사이즈로 있다.**

1. puts
    `int puts(const char *s);`

    출력 후 자동 개행

2. fputs

    `int fputs(conts char *s, FILE *stream);`

    출력 후 개행 없음

### 예제_2

```c
# include <stdio.h>

int main(void){
    char text[] = "Simple String" // 문자열은 이미 배열 포인터
    char *str;
    str = text // str에는 주소가 들어올 수 있고 text는 이미 배열 포인터이다.

    printf("1. puts tests ---- \n");
    puts(str);
    puts("So Simple string!");

    printf("2. fputs tests ---- \n");
    fputs(str, stdout);
    printf("\n");
    fputs("So Simple String2!", stdout);
    printf("\n");

    printf("3. end of main ----");
    return 0;
}

/*
console -->
1. puts tests ----
Simple String
So Simple string!
2. fputs tests ----
Simple String
So Simple String2!
*/
```

* 문자열 입력 함수

* gets

    `char *gets(char *s);`

    ```c
    int main(void){
        char str[7]; // 7바이트 메모리 문자배열 할당
        gets(str); // 입력 받은 문자열을 배열 str에 저장
    }
    ```

미리 마련해 놓은 배열을 넘어서 문자열이 입력되면, 할당 받지 않은 할당 받지 않은 메모리 공간을 침범하여 오류가 발생.

따라서 가급적이면 아래의 형태로 `fgets`함수를 호출 하는 것이 좋다.

* fgets

    `char *fgets(char *s, int n, FILE *stream);`

    ```c
    char str[7];
    fgets(str, sizeof(str), stdin); // stdin으로부터 문자열 받아 str에 저장.
    ```

    stdin으로부터 문자열을 받아 배열 `str`g함수에 저장화되, sizeof(str)의 길이 만큼만 저장해라.

    이럴 경우 stdin에 10바이트가 들어왔으면,
    앞에서부터 7바이트만 끊어서 저장하게 된다.

    1. `"123456789"`를 입력.
    2. `"123456"`이 저장. 마지막 1바이트는 NULL문자.

___

## 표준_입출력과_버퍼

[i3]: #표준_입출력과_버퍼

ANSI C의 표준에서 정의된 함수들을 표준 입출력 함수라 한다.

`printf`, `scanf`, `fputc`, `fgetc`, `fputs`, `fgets` 모두 표준 입출력 함수이다.

이 표준 입출력 함수를 통해서 입출력 하는 경우, 해당 데이터들은 운영체제가 제공 하는 **`메모리 버퍼`**를 중간에 통과하게 된다.

`메모리 버퍼`는 디바이스 드라이버에 있으며, 커널과 연계된 데이터를 임시로 모아두는 메모리 공간이다.

### 버퍼링을 하는 이유는 무엇 인가?

1. 물론, 키보드 디바이스는 캐릭터 디바이스로서 단일 문자가 입력되면, 바로 인터럽션이 일어나지만,
2. 버퍼링을 통해서 그것을 저장해놓는 등으로 해당 인터럽션을 중요도가 낮은 인터럽션으로 처리하는 행동을 한다.
3. 그리고 엔터키가 입력되었을떄. 읽어들인다는 행동으로 연결지어,
4. 저장해놓았던 버퍼를 소모하고 메모리를 비운다.

이 버퍼링의 가장 큰 이유는 **데이터 전송의 효율성**과 관련이 있다.

키보드나 모니터 같은 외부장치의 데이터 입출력은 생각보다 시간이 오래 걸리는 작업이다.  
따라서 버퍼링 없이 키보드가 눌릴때마다, 바로 목적지 프로세스로 이동시키는 것보다.  
중간에 메모리 버퍼를 둬서 데이터를 한데 묶어 이동시키는 것이 효율적이고 빠르다.

### 출력 버퍼를 비우는 `fflush`함수

**함수호출 성공 시 0, 실패시 EOF 반환**

`int fflush(FILE *stream)`

`fflush(stdout)`으로 호출하면, 표준 출력버퍼를 비워라 라는 명령이다.

___

### 입력버퍼 비우기

출력버퍼를 비우는 것이 데이터가 목적지(프로세스)로 전송됨을 의미한다면,  
입력버퍼를 비우는 것은 **데이터의 소멸**을 의미한다.

* 예제_1

```c
#include <stdio.h>

int main(void){
    char perID[7];
    char name[10];

    fputs("주민번호 앞 6자리 입력: ", stdout);
    fgets(perID, sizeof(perID), stdin); // block IOWAIT

    fputs("이름 입력: ", stdout);
    fgets(name, sizeof(name), stdin); // block_IOWAIT

    printf("주민번호: %s\n", perID);
    printf("이름: %s\n", name);
    return 0;
}
/*
console -->
주민번호 앞 6자리 입력: 231423
이름 입력: 주민번호: 231423
이름: 

*/
```

1. 첫 `fgets()` 에서 7바이트를 읽어들이라고 했지만, \n을 만나는 순간 읽지 못하고 6바이트만 읽어들임.
2. 2번째 `fputs()`에서 \n이 버퍼에 있기 떄문에, 바로 이스케이프.

* 예제_1 개선

```c
#include <stdio.h>

void ClearLineFromReadBuffer(void){
    while(getchar()!= '\n');
}

int main(void){
    char perID[7];
    char name[10];

    fputs("주민번호 앞 6자리 입력: ", stdout);
    fgets(perID, sizeof(perID), stdin); // block IOWAIT
    // buffer = '\n'
    ClearLineFromReadBuffer(); // 입력버퍼 비우기
    // buffer = ''

    fputs("이름 입력: ", stdout);
    fgets(name, sizeof(name), stdin); // block_IOWAIT
    ClearLineFromReadBuffer(); // 입력버퍼 비우기

    printf("주민번호: %s\n", perID);
    printf("이름: %s\n", name);
    return 0;
}
```

`ClearLineFromReadBuffer`에서 한문자로 취급되는 `\n` null이 읽혀지면 while문을 더 이상 수행하지 않는다.

따라서 그 시점에 버퍼에서 \n이라는 문자가 읽혀짐으로써 버퍼의 첫 \n이 사라짐.
___

## FILE과_STREAM_그리고_일반파일의_입출력

[i4]: #FILE과_STREAM_그리고_일반파일의_입출력

### fopen함수 호출을 통한 파일과 스트림 형성, FILE구조체

**성공 시 해당파일의 FILE구조체 변수의 주소 값, 실패시 NULL포인터 반환**

`FILE fopen(const char *filename, const char *mode);`

```c
#include <stdio.h>

int main(void)
{
    FILE * fp=fopen("simple.txt", "wt");
    if(fp==NULL){
        puts("file open fails!");
        return -1;
    }

    fputc('A', fp);
    fputc('B', fp);
    fputc('C', fp);
    fputs("sample!\n", fp);
    fclose(fp);
    return 0;
}
```

`wt`모드로 inode를 가르키고 close하는 순간 실행중인 프로세스 메모리가 해제된다.

fputc를 실행한다고, 바로 저장되는 것이 아니라,  
운영체제 단에서는 어느정도 버퍼메모리에 저장해놓고, 모든 변경된 버퍼를 수집해서 최적수준으로 정려한 후에 디스크에 쓴다.  
이러한 과정을 `writeback`이라 한다.

이런 방식은 쓰기호출을 빠르게 수행해서 거의 즉시 반환하도록 만든다.  

그래서 버퍼메모리에 있지만 아직 파일 HDD로 저장하겠다는 시스템콜로 전달되지 않았다면.  
중간에 있던 커널쪽에 프로세스별로 디바이스 드라이버를 통해 관리하던 버퍼가 소멸되면서, 저장되지 못한채로 끝나게 된다.  

그런 문제를 방지 하기 위해서, 커널은 최대 버퍼나이를 만들어 나이가 꽉찬 변경된 버퍼를 빠짐없이 기록한다.  
물론 HDD에 전달하였으나, HDD에서 또한 버퍼메모리로 잡아놓고 실제 물리적으로 저장하는 것은 좀 더딘 일이 되기도 한다.

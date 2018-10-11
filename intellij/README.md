# IntelliJ

인텔리 쓰다가 신박한 기능같은거 있으면 기록하려고

### .http 를 이용하여 Postman 대체하기

#### 사용방법
```
아무 프로젝트 내에서 .http 파일을 생성하면 됨

[마우스 우클릭] - [new] - [HTTP Request] - [파일명 입력]
```

하고 나면 파일명.http 파일이 생성

여기에 HTTP request 를 직접 날리는 느낌 비슷하게 아래 내용 기술 (예)
```
GET http://blog.daum.net
```

입력 후 커서를 것다대고 Option + Enter = Run

```
GET http://blog.daum.net

HTTP/1.1 200 OK
Date: Thu, 11 Oct 2018 04:11:40 GMT
Server: Apache
Vary: User-Agent,Accept-Encoding
Accept-Ranges: bytes
Expires: Fri 01 Jan 1990 00:00:00
Cache-Control: no-cache,must-revalidate
Pragma: no-cache
Keep-Alive: timeout=1, max=10
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<meta http-equiv="X-UA-Compatible" content="IE=IE7" />
		<meta name="description" content="인터넷의 또 다른 세상, Daum 블로그" />
		<meta name="keywords" content="블로그, 다음 블로그, Daum 블로그, 다음, Daum, 생각, 글, 공간, 다음다섯, 행복한 사진관, 블로거 기자단, 블로그 재발견, 인기 최고" />
        	<link rel="icon" type="image/ico" href="//blog.daum.net/favicon.png" />

... 이하 생략
```

이런식으로 아래 코드 하이라이팅까지 돼서 간단하게 보기 좋은 모양으로 나옴


POST 요청은 뭐 이런식(다른 메소드도 예상한바대로 쓰심되고)
```
POST http://blog.daum.net
```

쿠키를 실어보내고 싶으면
```
POST http://blog.daum.net
Cookie: key=value
```

요청했을때 로그도 쌓이고 있는데 경로는
```
{PROJECT_ROOT}/.idea/httpRequest/
```


스프링 사용시 컨트롤러에 이 .http 를 자동 생성해주는 기능이 있음(신박)
```
컨트롤러 어노테이션 옆에 스프링 빈 모양이 있는곳 클릭해주면 생성 옵션이 나옴.. 신기~~
```


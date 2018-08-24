# Nginx 기초 사용법 정리
nginx의 기본 환경 설정법에 대한 정리를 다룹니다.  
conf 파일에 들어가는 기본적인 문법(?), Directives(지시자)들의 기초적인 사용법을 소개합니다.  
nginx 공홈의 beginner's guide위주로 정리  

## Nginx
- **Reactor** 패턴
  - 하나는 Event를 받고 전달해 주는 리액터
  - 다른 하나는 리액터가 보낸 Event를 실제로 받아 처리하는 핸들러
    - one master process(conf, worker 관리)
    - several worker processes (실제 요청 처리)
- 기본적으로 single thread를 이용해서, 몇 천개의 connection도 효율적으로 관리가 가능하다고 함
  - 필요에 따라 fork를 써서 몇 개의 process 사용 가능
- Main Event Loop라는 놈이 OS의 socket으로 부터 읽을 수 있는 Data를 기다리고 있음

## 설치는 알아서

## Command
- stop, quit, reload, reopen 등이 있음
- 이놈들을 이용해서 nginx 자체를 키고 끄고 conf를 리로드하는 등의 컨트롤을 할 수 있습니다.
```
  nginx -s 커멘드
```

## Conf 파일
- 저는 보통 /etc/nginx/site-availables/default 파일을 수정함으로써 Nginx 세팅을 합니다
- nginx가 설치되면 자동으로 생성됨

## Log 파일
- 아마 /log/nginx에 보면
- access.log, error.log가 있을 것인데
- tail -f 명령어로 실시간 로그를 찍어보는 식으로 활용합니다

## Directives(지시어)
- nginx는 **directives에 의해 컨트롤되는 모듈**로 이루어져있습니다.
- directive의 형식에는 두 가지가 있습니다.
  - **simple** directive
    ```
    directive_name params
    ```
  - **block** directive
    ```
    directive_name {
      ~~~
    }
    ```
  - 만약 block directive의 { } 안에 다른 directive가 있다면 block을 안에 있는 놈들의 **context**라고도 부르기도 합니다.
  - 참고로, 어디에도 포함되지 않은 directive는 main context에 포함되었다고 말합니다.

## Conf 기본 예시
```
server { # 하나의 웹사이트 선언
  listen 80; # 리스닝 포트
  server_name abc.zum.com;
  
  location / { # 특정 URL 처리
    root /home/nginx; 
    index index.html, index.htm; # 초기 페이지 설정
  }
  
  location ~ \.do$ { # 특정 확장자 요청 넘기기 (nginx 뒷단의 WAS로)
    proxy_pass http://localhost:8080; # 
  }
}
```
- 대충 이런식으로 생겨 먹었습니다.
  - **server** 블록은 **하나의 웹사이트를 선언**할 때 쓰입니다.
  - **location** 블록은 server 블록 안에 등장하고, **특정 URL을 처리하는 법을 정의** 합니다. (뒤에서 자세히 설명)
    - 위 예시에선 루트(/)로 접속 했을 때, 
      /home/nginx에 있는 index.html을 서빙하는 의미겠죠?
    - 두 번째 location 블록은 특정 패턴을 명시하고 있습니다.  
      '.do'로 끝나는 주소 요청은 로컬호스트의 8080포트로 넘어갑니다.
      - 이런 패턴은 웹서버 뒤에 WAS(웹애플리케이션서버)가 있어서,  
        요청을 받은 웹서버가 WAS로 요청을 넘길 때 씁니다.

## "Serving Static Content"은 어떤 과정을 거쳐 처리되는가?
- nginx는 웹서버로 static content(html, css, image, ...)를 서빙하는 용도의 소프트웨어입니다.
```
  http { # 생략 가능한 듯
    server {
      listen ...
      server_name ...
      location ...
    }
    
    server {
      listen ...
      server_name ...
      location ...
    }
  }
```
- 일반적으로 conf는 여러 server block으로 이루어져있고,
- server block은 **listen하고 있는 port와 server name**에 의해 구분됩니다.
  1. 일단 어떤 server block으로 요청을 처리시킬지 결정하게 되면,
  2. 그 안에 있는 location 지시어를 참조해 URI 매칭을 시도하는 식으로 흘러갑니다!

## location directive 사용법
- Server block 안에서 URI 매칭을 하는 친구
- 주소 매칭을 할 때, 당연히 regular expression도 쓸 수 있습니다. (알아서 찾아보기 - 나도 헷갈림)
```
server {
  location / {
    root /data/www;
  }
  
  location /images/ {
    root /data;
  }
}
```
1. URI가 / 와 매칭된다면, root에 명시된 주소와 URI를 합칩니다.
  - 만약 매칭 후보가 여럿이라면 가장 긴 prefix에 해당하는 놈을 고릅니다.
2. /images/로 시작하는 URI라면
   서버는 /data/images 디렉터리로 부터 파일을 전송합니다. (없으면 404 status code)
  - ex) http://localhost/images/example.png 는 **/data/images/example.png**로 바뀜
cf) /images/로 시작 안되는 것들은 모두 /data/www/로 매핑
  - ex) http://localhost/some/example.png 는 **/data/www/some/example.png**로 바뀜 

## Virtual Host, Sub Domain 설정법
- 하나의 서버 안에 2개의 웹사이트를 선언?
- conf 파일에 server 블록 자체를 별도로 추가하면 가능합니다. 
```
server {
  listen 80;
  server_name labs.zum.com;
  ~~~~
}
```

## 용어 정리

### Proxy Server ?
- 클라이언트를 자신을 통해서 다른 네트워크 서비스로 중계해주는 소프트웨어
- Proxy Server를 둠으로써 뒤에 분산 시스템을 숨길 수 있음
- "revere proxy"로 부르기도

### Upstream ?
- Origin 서버
  - 여러 대의 컴퓨터가 순차적으로 어떤 일을 할 때, 어떤 서비스를 받는 서버를 의미한다.
- Nginx 자체는 Down Stream 서버라고 한다.
- **upstream은 nginx의 내장모듈**로서, **부하 분산(로드 밸런싱) 및 속도 개선**에 쓰인다. 사용법도 매우 간단데스

# 참고
- [nginx beginner's guide](http://nginx.org/en/docs/beginners_guide.html)
- [nginx 주요 설정 / 필독](https://sarc.io/index.php/nginx/61-nginx-nginx-conf)
- [nginx 공홈에서 공개한 쿡북 / 매우 유용](https://www.nginx.com/resources/library/complete-nginx-cookbook/)
  - speed up options ; 4page, reset_timeout_connections on; 및 gzip 사용..
- [생활 코딩 nginx / 걍 참고용](https://opentutorials.org/module/384/3463)

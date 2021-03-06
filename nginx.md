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

## 설치
- 
  ```bash
  sudo apt-get update
  sudo apt-get install nginx
  ```

## Command
- stop, quit, reload, reopen 등이 있음
- 이놈들을 이용해서 nginx 자체를 키고 끄고 conf를 리로드하는 등의 컨트롤을 할 수 있습니다.
```
  nginx -s 커멘드
```

## Conf 파일
- 보통 /etc/nginx/site-availables/default 파일을 수정함으로써 Nginx 세팅을 하거나 
  - /etc/nginx/conf.d/default.conf 을 수정해서 세팅합니다.
  - 버전에 따른 차이인 것 같습니다. 1.10.x 버전에는 전자, 1.15.x버전은 후자였음 
  - 설치 리포지토리가 달라지면 다를 수
- nginx는 기본적으로 /etc/nginx/nginx.conf를 기본 conf 파일로 로드하고,
  - nginx.conf는 http block을 정의하고, http 서버의 전반적인 설정이 추가되어 있음
  - nginx.conf 안에 보면 include /etc/nginx/conf.d/\*.conf 와 같이 
    별도의 어쩌구.conf 를 포함하는 구조로 되어있음
      - 이렇게 포함되는 conf에서는 server block(virtual host)에 관한 설정이 추가되어 있음
    ```
    http {
      include /etc/nginx/conf.d/*.conf;
      ~~~
    }
    ```
  - 따라서 /etc/nginx/nginx.conf를 확인하신 후, 
    include 되는 디렉터리에 있는 default 파일을 수정하시거나, \*.conf 파일을 따로 생성해서 작성합니다.
  - [참고, site-available, conf.d 차이를 알고 싶을 때](https://serverfault.com/questions/527630/what-is-the-different-usages-for-sites-available-vs-the-conf-d-directory-for-ngi)

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

## Proxy Server 설정법
- 클라이언트를 자신을 통해서 다른 네트워크 서비스로 중계해주는 소프트웨어
  - Receive requests and pass them to the proxied servers
  - Retrieves responses from them and send them to the client
- Proxy Server를 둠으로써 뒤에 분산 시스템을 숨길 수 있음
- "Revere Proxy"로 부르기도
```
server {
  listen 8080;
  root /data/up1; # maps all requests to the /data/up1 (local file system)
                  # server의 context 속에 위치할 수도 있음 (location에 root 지시어가 따로 없다면)
  location / {
  }
}
server {
  location / {
    proxy_pass http://localhost:8080;
  }
  location ~ \.(gif|jpg|png)$ {
    root /data/images;
  }
}
```
- regex는 ~로 시작한다.(좌우에 띄워줘야 하는 듯)
- 두번째 location에 나온 패턴은
  - filter requests ending with .gif, .jpg, .png and map them to /data/images/ directory
    by adding URI to the root directive param
- 그 외의 패턴은 모두 8080(proxied server)으로 pass된다.

- nginx가 location블락을 선택하는 기준은 다음과 같다.
- **When Nginx select a location block to server a request...**
  1. checks location directives param & remember the longest prefix
  2. checks regex
      - if there is mached one -> pick that location
      - else, pick remembered one

## Cache Server 설정법
- [nginx-by-examples-caching](http://bitsandpieces.it/nginx-by-examples-caching)
- [reverse-proxy-with-cache](https://www.nginx.com/resources/wiki/start/topics/examples/reverseproxycachingexample/)

## Upstream ?
- Origin 서버라고도 불림
  - 여러 대의 컴퓨터가 순차적으로 어떤 일을 할 때, 어떤 서비스를 받는 서버를 의미한다.
- Nginx 자체는 Down Stream 서버라고 한다.
- **upstream은 nginx의 내장모듈**로서, **부하 분산(로드 밸런싱) 및 속도 개선**에 쓰인다. 사용법도 매우 간단데스
  - [nginx를 이용한 load balancing 예제](https://www.nginx.com/resources/wiki/start/topics/examples/loadbalanceexample/)
- [upstream / downstream 개념](https://stackoverflow.com/questions/32364579/upstream-downstream-terminology-used-backwards-e-g-nginx)

# 참고
- [nginx beginner's guide](http://nginx.org/en/docs/beginners_guide.html)
- [nginx 주요 설정 / 필독](https://sarc.io/index.php/nginx/61-nginx-nginx-conf)
- [nginx 공홈에서 공개한 쿡북 / 매우 유용](https://www.nginx.com/resources/library/complete-nginx-cookbook/)
  - speed up options ; 4page, reset_timeout_connections on; 및 gzip 사용..
- [생활 코딩 nginx / 걍 참고용](https://opentutorials.org/module/384/3463)
- upstream 혹은 proxy 설정을 할 경우 소켓에 대한 얘기가 나오는데 참조하자  
  - [네트워크 소켓 개념](http://eastroot1590.tistory.com/entry/socket-socket%EC%9D%B4%EB%9E%80)  
  - [유닉스 소켓 vs TCP/IP host:port](https://serverfault.com/questions/195328/unix-socket-vs-tcp-ip-hostport)  
  - [유닉스 소켓](http://wiki.nex32.net/%EC%9A%A9%EC%96%B4/%EC%9C%A0%EB%8B%89%EC%8A%A4_%EB%8F%84%EB%A9%94%EC%9D%B8_%EC%86%8C%EC%BC%93)

- [nginx 설정 - 정적파일캐싱,WAS연동](http://areumgury.blogspot.com/2016/08/nginx.html)
- [기본적인 보안 설정](https://medium.com/daidalos-hoho/nginx-%EA%B8%B0%EB%B3%B8%EC%A0%81%EC%9D%B8-%EB%B3%B4%EC%95%88-%EC%84%A4%EC%A0%95-c239c3315ef8)

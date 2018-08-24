# Nginx 기초 사용법 정리
nginx의 기본 환경 설정법에 대한 정리를 다룹니다.  
conf 파일에 들어가는 기본적인 문법(?) block들의 의미에 대한 기초적인 사용법을 소개합니다.  
nginx 공홈의 beginner's guide위주로 정리  

## 설치는 알아서

## conf 파일
- 저는 보통 /etc/nginx/site-availables/default 파일을 수정함으로써 Nginx 세팅을 합니다

## Block
- conf 파일을 이루는 요소들을 블록이라 불렀던 것 같음
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
  - **server** 블록은 <U>하나의 웹사이트를 선언</U>할 때 쓰입니다.
  - **location** 블록은 server 블록 안에 등장하고, <U>특정 URL을 처리하는 법을 정의</U> 합니다.
    - 위 예시에선 루트(/)로 접속 했을 때, /home/nginx에 있는 index.html을 서빙하는 의미겠죠?
    - 두 번째 location 블록은 특정 패턴을 명시하고 있습니다. '.do'로 끝나는 주소 요청은 로컬호스트의8080 으로 넘어갑니다.
      - 이런 패턴은 웹서버 뒤에 WAS(웹애플리케이션서버)가 있어서,  
        요청을 받은 웹서버가 WAS로 요청을 넘길 때 씁니다.

## Virtual Host, Sub Domain
- 하나의 서버 안에 2개의 웹사이트를 선언?
- conf 파일에 server 블록 자체를 별도로 추가하면 가능합니다. 
```
server {
  listen 80;
  server_name labs.zum.com;
  ~~~~
}
```


# 참고
- [nginx beginner's guide](http://nginx.org/en/docs/beginners_guide.html)
- [nginx 주요 설정 / 필독](https://sarc.io/index.php/nginx/61-nginx-nginx-conf)
- [nginx 공홈에서 공개한 쿡북 / 매우 유용](https://www.nginx.com/resources/library/complete-nginx-cookbook/)

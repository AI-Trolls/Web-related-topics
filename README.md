# 웹 관련 토픽

웹서버 및 자주쓰는 flask, node에 관한 개념, 용어들, 배포시 알아둬야할 점들 정리

## Web Server (주의; 웹서버와 웹애플리케이션서버는 다른 개념임)
- 클라이언트로부터 요청된 웹 리소스을 서빙해주는 소프트웨어
- **기본적으로 정적(static)한 자원을 그대로 리턴해주는 역할**
  - 동적인 컨텐츠를 서빙하고 싶다면 뒷단에 WAS(flask, node, rails ...)를 두고   
    별도로 커뮤니케이션함으로써 컨텐츠를 제공한다.
- **80번 포트** 리스닝
  - 80번에서 듣고있다가 요청이 들어오면 해석해서 응답
  - CGI라는 것을 통해 앱을 실행시키거나, static file 리턴
- **nginx, apache** 서버 등이 있음
  - 프로세스 기반의 apache 서버도, 최근에는 쓰레드를 이용해 nginx와 비슷한 성능 보인다함
  - [Nginx 기초 사용법 정리](https://github.com/AI-Trolls/Web-related-topics/blob/master/nginx.md)

## WAS
- **Web Application Server**
- 웹 서버 뒤에 붙는 애플리케이션 서버(flask, node, rails...)

## CGI
- **Common Gateway Interface**, 프로토콜 개념임
- 정적(static)인 웹 서버에서 앱(dynamic)을 작동시키기 위한 인터페이스
- **웹 서버와 외부 애플리케이션(WAS) 사이의 인터페이스**가 **CGI**
- 기존에는 요청이 들어오면 CGI를 통해 외부 프로그램을 실행(fork)시켜 응답을 했지만
  - 모든 요청마다 sub-process fork
- 근래에는 인터프리터를 웹서버 자체에 내장시켜 따로 프로세스를 fork시키지 않고 내부에서 처리하기도 함

## WSGI
- **Web Server Gateway Interface**의 약자
- 웹서버(Nginx)에서 받은 요청에 대한 동적인 처리가 필요한 경우, 
  wsgi가 서버사이드 애플리케이션(flask)을 실행
- CGI 디자인 패턴에 기반한 인터페이스, **Python**에 종속된 개념
- **파이썬 스크립트가 웹 서버와 통신하기 위한 프로토콜**
- flask, django같은 프레임워크는 여러 요청을 concurrently하게 처리하게끔 설계되지 않음. 
  - **WSGI middleware(server)는 이 부분을 처리함** 
- 웹서버와 파이썬애플리케이션 사이에 존재하는 개념이라고 생각하면 됨
  - WSGI **middleware**
    - WSGI 요청을 처리하려면 웹 서버에서  정보 및 Callback 함수를 애플리케이션에 제공해야함
    - 애플리케이션은 요청 처리 후 응답을 Callback을 통해 처리
    - WSGI middleware란 놈이 이러한 작업들을 처리해줌
    - 이 미들웨어는, 웹서버의 관점에서는 애플리케이션이고, 파이썬 애플리케이션 관점에선 서버로서 행동
    - 관계 : [웹서버] **<----->** [미들웨어] **<----->** [파이썬 애플리케이션]
    - WSGI 규격에 맞춘 코드를 실행해주는 프로그램으로는 uWSGI, gunicorn 등이 있음 
    - Application Container로도 불림
- [Python WSGI의 역사 및 개념](https://blog.appdynamics.com/engineering/an-introduction-to-python-wsgi-servers-part-1/)


# 파이썬 Production mode 배포 관련

## flask 
- 내부적으로 blocking 작업이 있을 경우
  - A가 해당 API 작업에 요청을 보내고, B가 동일 API에 요청을 해도, A가 다 처리 될 때까지 기다림
  - 따라서 flask 앱 내부에서 IO작업(request, file i/o)를 할 때 주의해야함
- flask 앱은 기본적으로 *Synchronous*하다.
  - **gunicorn**을 통해서 여러 요청의 동시 처리를 가능케할 수 있다.
  - 필요에따라 지정한 만큼 인스턴스(workers)를 몇개 만들어 동시 서비스를 제공하게끔 동작한다.
  - [gunicorn](http://gunicorn.org)
  - [supervisor](http://supervisord.org/index.html) ; node의 forever같은 것
    - python2만 정식 지원이지만
    - python3도 dev버전을 통해 쓸 수 있음(아래 설치법)
    ```
    pip install git+https://github.com/Supervisor/supervisor
    ```
    - [pip으로 설치하는 경우 인스트럭션](http://supervisord.org/installing.html#installing-via-pip)
    - [사용법 문서](http://supervisord.org/running.html)
      - 간단히 정리하면, pip으로 설치 이후에
        1. /etc/supervisord.conf를 만들고(위 링크 참조) 
        2. 내가 실행하고 싶은 프로그램 설정을 supervisord.conf에 추가 (concat)
          ```
          [program:myapp]
          directory=/root/myapp
          command=gunicorn app:app -b 0.0.0.0:22222
          autostart=true
          autorestart=true
          stderr_logfile=/root/myapp/err.log
          stdout_logfile=/root/myapp/out.log
          ```
        3. 이후에 supervisord(-c옵션)를 실행하는 것으로 충분하다. (pip으로 설치한 경우에 얘기)
      - supervisord 실행 이후에 conf가 바뀌는 경우엔 아래와 같이(?)
        - supervisorctl은 supervisor에 명령을 보낼 수 있는 커멘드라인 툴
        ```
          supervisorctl reread #를 치면 available한 프로그램 이름이 뜨고
          supervisorctl update #까지 하면 available 했던 프로그램이 프로세스 그룹에 추가됐다고 뜸
        ```
    - docker와 함께 쓸 경우 주의해야 할점은, **supervisor를 foreground에서 실행**할 필요가 있을 수 있다는 것
      - **해당 경우엔,** supervisord.conf의 nodaemon옵션을 true로 바꿔준다. (start in foreground if true)
    - 기본 문서를 보고 잘 안된다면 아래 문서 참조 (apt-get install 방식으로 깐 경우 conf가 자동 생성되는 듯)
      - [conf에 프로그램 등록한 이후, 프로세스 추가 및 실행하는 법](https://serversforhackers.com/c/monitoring-processes-with-supervisord)
      - [비슷한 글 - 좀 더 깔끔?](https://www.vultr.com/docs/installing-and-configuring-supervisor-on-ubuntu-16-04)
      - [쉽게 설명된 블로그글](https://yenoss.github.io/2017/09/10/supervisor/)
      - [쉽게 설명된 블로그글2](https://jwkcp.github.io/2016/11/07/how-to-use-supervisor-in-one-minute/)
      - [docker와 같이 쓸 때 참고](https://blog.trifork.com/2014/03/11/using-supervisor-with-docker-to-manage-processes-supporting-image-inheritance/)
- [flask-gunicorn 예제1](https://medium.com/ymedialabs-innovation/deploy-flask-app-with-nginx-using-gunicorn-and-supervisor-d7a93aa07c18)
- [flask-gunicorn 예제2](http://egloos.zum.com/mcchae/v/11149241)
- [flask-gunicorn, logging](https://medium.com/@trstringer/logging-flask-and-gunicorn-the-manageable-way-2e6f0b8beb2f)
- gunicorn shared memory between instance 로 구글링 해보기

## 참고
- [gunicorn으로 flask 동시 요청 처리](https://winterj.me/flask-concurrency-test/)
- [uwsgi를 통한 python3, nginx 배포](https://mango-tree.github.io/2017/03/27/uWSGI-%EC%99%80-Python3%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-Nginx%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)
- [uwsgi를 통한 flask, nginx 배포](https://cjh5414.github.io/flask-uwsgi-nginx/)
- [웹서버, WAS, CGI 개념](http://khanrc.tistory.com/entry/%EC%9B%B9%EC%84%9C%EB%B2%84-WAS-CGI) 
- [WSGI로 보는 웹 서버 개념](http://khanrc.tistory.com/entry/WSGI%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%9D%98-%EA%B0%9C%EB%85%90)
- [WSGI와 CGI의 차이](http://khanrc.tistory.com/entry/WSGI%EC%99%80-CGI%EC%9D%98-%EC%B0%A8%EC%9D%B4)


# 노드 관련 토픽
- mongoose를 이용한 mongoDB 사용법 ; https://velopert.com/594

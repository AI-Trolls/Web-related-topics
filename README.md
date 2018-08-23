# 웹 관련 토픽

웹서버 및 자주쓰는 flask, node에 관한 개념, 용어들, 배포시 알아둬야할 점들 정리

## Web Server (주의; 웹서버와 웹애플리케이션서버는 다른 개념임)
- 클라이언트로부터 요청된 웹 리소스을 서빙해주는 소프트웨어
- 기본적으로 정적(static)한 자원을 그대로 리턴해주는 역할
- 80번 포트 리스닝
  - 80번에서 듣고있다가 요청이 들어오면 해석해서 응답
  - CGI라는 것을 통해 앱을 실행시키거나, static filer 리턴
- **nginx, apache** 서버 등이 있음(최근에는 apache서버도 쓰레드를 이용해 nginx와 비슷한 성능 보인다함)

## WAS
- Web Application Server
- 웹 서버 뒤에 붙는 애플리케이션 서버(flask, node, rails...)

## CGI
- Common Gateway Interface, 프로토콜 개념임
- 정적(static)인 웹 서버에서 앱(dynamic)을 작동시키기 위한 인터페이스
- 웹 서버와 외부 애플리케이션 사이의 인터페이스가 **CGI**
- 기존에는 요청이 들어오면 CGI를 통해 외부 프로그램을 실행(fork)시켜 응답을 했지만
  - 모든 요청마다 sub-process fork
- 근래에는 인터프리터를 웹서버 자체에 내장시켜 따로 프로세스를 fork시키지 않고 내부에서 처리한다고 함.

# WSGI
- CGI 디자인 패턴에 기반한 인터페이스, **Python**에 종속된 개념
- CGI 처럼 모든 요청에 대해 sub-process를 fork하지 않음
- 파이썬 스크립트가 웹 서버와 통신하기 위한 프로토콜
- 웹서버와 파이썬애플리케이션 사이에 존재하는 개념이라고 생각하면 됨
  - WSGI middleware
    - WSGI 요청을 처리하려면 웹 서버에서 환경 정보 및 Callback 함수를 애플리케이션에 제공해야함
    - 애플리케이션은 요청 처리 후 응답을 Callback을 통해 처리
    - WSGI middleware란 놈이 이러한 작업들을 처리해줌
    - 웹서버의 관점에서는 애플리케이션이고, 파이썬 애플리케이션 관점에선 서버로서 행동
    - [웹서버] ----- [미들웨어] ----- [파이썬 애플리케이션]
    - mod_wsgi, uwsgi, gunicorn, tornado 등이 있다.
    - Application Container로도 불림

# 파이썬 Production mode 배포 관련

## flask 
- 내부적으로 blocking 작업이 있을 경우
  - A가 해당 API 작업에 요청을 보내고, B가 동일 API에 요청을 해도, A가 다 처리 될 때까지 기다림
  - 따라서 flask 앱 내부에서 IO작업(request, file i/o)를 할 때 주의해야함
- flask 앱은 기본적으로 *Synchronous*하다.
  - **gunicorn**을 통해서 여러 요청의 동시 처리를 가능케할 수 있다.
  ```
  gunicorn flask_app:app -w $NUM_WORKER --threads $NUM_THREADS -k $WORKER_CLASS
  ```

## WSGI?
- **Web Server Gateway Interface**의 약자
- 웹서버(Nginx)에서 받은 요청에 대한 동적인 처리가 필요한 경우, 
  wsgi가 서버사이드 애플리케이션(flask)을 실행
- WSGI 규격에 맞춘 코드를 실행해주는 프로그램으로는 uWSGI, gunicorn 등이 있음 (똑같이 pip으로 설치하여 사용)

## 참고
- [gunicorn으로 flask 동시 요청 처리](https://winterj.me/flask-concurrency-test/)
- [uwsgi를 통한 python3, nginx 배포](https://mango-tree.github.io/2017/03/27/uWSGI-%EC%99%80-Python3%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-Nginx%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)
- [uwsgi를 통한 flask, nginx 배포](https://cjh5414.github.io/flask-uwsgi-nginx/)
- [웹서버, WAS, CGI 개념](http://khanrc.tistory.com/entry/%EC%9B%B9%EC%84%9C%EB%B2%84-WAS-CGI) 
- [WSGI로 보는 웹 서버 개념](http://khanrc.tistory.com/entry/WSGI%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%9D%98-%EA%B0%9C%EB%85%90)
- [WSGI와 CGI의 차이](http://khanrc.tistory.com/entry/WSGI%EC%99%80-CGI%EC%9D%98-%EC%B0%A8%EC%9D%B4)

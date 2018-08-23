# 웹 관련 토픽

## Nginx
- 웹서버
- 원칙적으로 Static한 파일만을 클라이언트에게 리턴해주는 역할

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
  wsgi가 서버사이드 애플리케이션을 실행
- WSGI 규격에 맞춘 코드를 실행해주는 프로그램으로는 uWSGI, gunicorn 등이 있음 (똑같이 pip으로 설치하여 사용)

## 참고
- [gunicorn으로 flask 동시 요청 처리](https://winterj.me/flask-concurrency-test/)
- [uwsgi를 통한 python3, nginx 배포](https://mango-tree.github.io/2017/03/27/uWSGI-%EC%99%80-Python3%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-Nginx%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/)
- [uwsgi를 통한 flask, nginx 배포](https://cjh5414.github.io/flask-uwsgi-nginx/)

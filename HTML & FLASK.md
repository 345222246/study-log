
## 웹 서버란?
- 요청을 받으면 응답을 돌려주는 프로그램이다. 브라우저의 요청을 처리해서 알맞은 웹 페이지를 반환하도록 논리 흐름이 구현된 프로그램이 바로 웹 서버입니다.
- 일반적으로 웹 서버는 웹 프레임워크를 사용해서 개발된다. 웹 프레임워크는 웹 개발을 위해 반복적이거나 기본적으로 필요한 기능과 뼈대를 제공하는 개발 환경이라 볼 수 있다. 
- 간단한 웹 페이지 하나를 제공하는 웹 서버를 구현할 때 웹 프레임워크의 도움을 받는다면 아래의 10줄 코드로도 구현 가능
<br>
```python

from flask import Flask, render_template   # Flask 웹 프레임워크 모듈을 불러옵니다.

app = Flask(__name__)

@app.route('/home')                        # /home 경로로의 요청이 들어오면,
def get_home():                            # get_home()을 실행합니다.
    return render_template('home.html')    # HTML 문서인 home.html을 반환합니다.

if __name__ == '__main__':
    app.run(host='0.0.0.0')                # 웹 서버를 구동합니다.
```
<br>
- flask는 python을 기반으로 하는 웹 프레임 워크 이다.
<br>
## flask 실습
### app.py 코드
<br>
```python
from flask import Flask, render_template          # Flask 웹 프레임워크 모듈을 불러옵니다.

app = Flask(__name__)

@app.route('/simple_page')                        # /simple_page 경로로의 요청이 들어오면,
def get_simple_page():                            # get_simple_page()를 실행합니다.
    return render_template('simple_page.html')    # HTML 문서인 simple_page.html을 반환합니다.

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=31337)           # 웹 서버를 구동합니다.
```
<br>
```python
from flask import Flask, render_template          # Flask 웹 프레임워크 모듈을 불러옵니다.
```
- python에서 flask모듈로 부터 flask 클래스와 render_template 함수를 가져오기.
- flask: flask 애플리케이션을 생성하는 데 사용되는 클래스.
- render_template: HTML 파일을 렌더링하여 클라이언트에게 반환하는 함수.
<br>
```python
app = Flask(__name__)
```
- flask 클래스의 인스턴스를 생성하여 app 변수에 할당.
<br>
```python
@app.route('/simple_page')                        # /simple_page 경로로의 요청이 들어오면, 
```
- python의 데코레이터라는 기능을 활용한 코드인데, /simple_page 경로로 들어오는 HTTP 요청을 바로 아래 위치한 함수인 get_simple_page()로 연결.
<br>
```python
def get_simple_page():                            # get_simple_page()를 실행합니다.
    return render_template('simple_page.html')    # HTML 문서인 simple_page.html을 반환합니다.
```
- 앞서 살펴본 데코레이터 @app.route('/simple_page')에 의해, /simple_page 경로로 요청이 들어오면 본 get_simple_page()가 실행됨.
- get_simple_page()는 simple_page.html이라는 이름을 render_template()으로 렌더링하여 이용자에게 반환.
<br>
```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=31337)           # 웹 서버를 구동합니다.
```
- flask 웹 서버 실행. host 인사를 '0.0.0.0'으로 설정함으로써 외부의 모든 이용자가 본 웹 서버에 접근할 수 있도록 설정. 
- port 인자를 31337로 설정하여 31337포트에 웹 서비스가 열리도록 지정.
<br>

### simplt_page.html 
```html
<!DOCTYPE html>
<html>
<head>
  <title>My Simple Web Page</title>
</head>
<body>
  <h1>Explore the World of HTML!</h1>
  <p>Greetings from the Web!</p>
</body>
</html>
```
<br>
### 결과 실행

>* Serving Flask app 'app'
>* Debug mode: off
>WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 >* Running on all addresses (0.0.0.0)
 >* Running on http://127.0.0.1:31337
 >* Running on http://192.168.160.187:31337
>Press CTRL+C to quit
<br>
>- Running on all addresses (0.0.0.0)
- flask 서버가 모든 공인 IP로부터 요청을 받아들인다는 의미. 다른 외부 네트워크 이용자가 본 웹 서버로 접속 가능.
<br>
>- Running on [http://127.0.0.1:31337](http://127.0.0.1:31337/)
- 로컬호스트에서도 웹 서버에 접근 가능하다는 의미. 웹 서버가 현재 실행중인 컴퓨터에서 접근 가능
<br>
>Running on [http://192.168.160.187:31337](http://192.168.160.187:31337/)
- 웹 서버의 특정 네트워크 인터페이스에 할당된 로컬 IP 주소인데, 쉽게 말해 로컬 네트워크 내에서도 본 웹 서버에 접근 가능하다는 의미.
<br>
## 템플린 렌더링
- flask에서는 render_template()을 사용해서 템플릿 렌더링을 할 수 있다. 
- **템플릿 렌더링**: 서버의 ptyhon 코드에서 생성한 변수나 값을 뼈대인 HTML 코드에 삽입하여 최종 HTML 문서를 만드는 과정
- **HTML 템플릿**: 뼈대인 HTML 코드
- **최종 HTML 문서**: HTML 템플릿에 실제 데이터가 채워져서 최종적으로 완성된 HTML 문서


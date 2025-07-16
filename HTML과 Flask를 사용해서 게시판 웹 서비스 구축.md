## 데이터 베이스
### DBMS
웹 서비스는 데이터베이스에 정보를 저장하고 이를 관리하기 위해 DBMS를 사용한다. DBMS는 데이터베이스에 새로운 정보를 기록하거나 기록된 내용을 수정, 삭제하는 역할을 한다. DBMS는 다수의 사람이 동시에 데이터베이스에 접근할 수 있고, 웹 서비스와 검색 기능과 같이 복잡한 요구사항을 만족하는 데이터를 조회할 수 있다는 특징이 있다. 

### RDBMS
RDBMS는 데이터를 표 형식으로 저장하고 관리하는 데이터 베이스 모델이다. 

### SQLite 실습
- 테이블 생성 명령어 - CREATE TABLE
- 고유키: 테이블의 특정 행을 고유하게 식별할 수 있는 필드로 설정하는 것
- 데이터 추가 명령어 - INSERT INTO
- 데이터 조회 명령어 - SELECT
- 데이터 수정 명령어 - UPDATE
- 데이터 삭제 명령어 - DELETE

### 로그인 기능 구현하기
- 다음 4개의 엔드포인트를 지원하도록 만들기
	- GET /login: 로그인 시도할 수 있는 웹 페이지 반환
	- POST /login: 로그인 시도를 실제로 처리함
	- GET /longin_success: 로그인 성공 시 본 페이지로 이동
	- GET /login_failure: 로그인 실패 시 본 페이지로 이동
- **리다이렉션**은 서버가 이용자에게 다른 URL로 이동하도록 명령하는 작업을 말한다.
- login.html
```html
<!DOCTYPE html>
<html>
<head>
  <title>Flask Simple Login</title>
</head>
<body>
  <h2>Login</h2>
  <form action="/login" method="post">
    Username:<br>
    <input type="text" id="username" name="username">
    <br><br>
    Password:<br>
    <input type="password" id="password" name="password">
    <br><br>
    <button>Login</button>
  </form>
</body>
</html>
```

### 세션이 적용된 로그인 기능
지금까지 만든 웹 서버의 가장 큰 문제는 로그인 상태가 유지되지 않는다는 점이다
이 문제를 해결하기 위해 세션을 도입할 수 있다
세션을 사용하면 서버가 이용자의 로그인 정보를 관리하고 이용자가 다른 페이지로 이동하거나 새로고침을 해도 서버가 해당 이용자의 인증 정보를 기억하고 있기 때문에 인증 상태를 유지할 수 있다
- app.py 변경사항

>-from flask import Flask, redirect, render_template, request, url_for<br>
>+from flask import Flask, redirect, render_template, request, session, url_for
>+import os

> ...
> app = Flask(__name__)
>+app.secret_key = os.urandom(32)
> ...
- 세션은 내부적으로 세션 내 저장된 데이터의 무결성을 보장하기 위해 암호학적인 비밀키를 필요로 하는데, 그 비밀키가 바로 **app.secret_key**이다. 이걸 설정하지 않으면 세션과 관련된 기능이 제대로 작동하지 않거나 보안 문제가 발생할 수 있다.

>session\['username']=username
- 이용자가 전달한 username과 password가 올바르면, session\['username']에 username을 대입
- 웹 서버의 session은 단순히 하나의 변수처럼 보이지만, flask는 내부적으로 세션 ID를 사용해 각 이용자의 데이터를 분리하여 관리하도록 되어 있음.
- 여러 이용자가 동시에 웹 서버를 이용하더라도 session이라는 변수에 담긴 데이터는 이용자별로 관리됨

### DB가 적용된 로그인 기능 만들기
저장해야할 데이터의 양이 많아지고 데이터의 구조가 복잡해지면 딕셔너리로는 관리하는 데에 한계에 부딪힘.

```python
def init_db():
	conn=sqlite3.connect('accounts.db')
	consor=conn.cursor()
	cursor.execute('''
		CREATE TABLE IF NOT EXITS accounts(
			id INTEGER PRINMARY KEY AUTOINCREMENT,
			username TEXT UNIQUE NOT NULL,
			password TEXT NOT NULL
		)
	```)
	conn.comit()
	conn.close()
```
- cursor는 데이터베이스에서 명령을 실행하고 처리하기 위한 인터페이스
- conn.commit: 데이터베이스의 변경사항을 저장

- flask 웹 실행부분에 데이터베이스를 초기화하는 init_db()를 실행하도록 추가.

### 회원가입 및 로그아웃 기능 구상
- **GET /register** 엔드포인트에 접근하면 아이디와 비번으로 회원가입을 시도할 수 있는 웹 페이지 반환
- **POST /register** 엔드포인트는 이용자가 GET /register에서 아이디와 비밀번호를 입력하고 회원가입 버튼을 누르면, 해당 아이디와 비번 데이터를 POST 메서드로 전달받아서 처리하는 역할.
- **GET /logout**

### 깔끔한 코드 패턴 만들기
위의 웹사이트는 이미 잘 작동하지만 기능끼리 연결성이 부족해서 이용하기 불편하고 확장성이 떨어지는 한계가 있다. 예를 들어서 이용자가 회원가입, 로그인, 로그아웃 기능을 이용하려면 **각각의 경로를 URL에 직접 입력해야만 접근**할 수 있다. 또한 **HTML코드가 중복으로 사용**되거나 데이터베이스 관련 코드가 웹 서버 코드에 몰려 있어서 **유지보수가 어려운 구조**를 띈다.
이를 해결하기 위해 각 주요 기능들을 보다 더 자연스러운 흐름으로 이용할 수 있도록 통합할 예정이다. 페이지마다 다른 기능을 사용할 수 있는 페이지로 이동할 수 있는 버튼을 만들어줄 것이다. 메인 페이지인 / 경로를 만들고, 해당 경로를 중심으로 웹 사이트를 재구성할 것이다.

- app.py 코드 수정
```python
@app.route("/",methods=['GET'])
def get_index():
	if 'username' not in session:
		return redirect(url_for('get_login'))

	return render_template('index.html',username=session['username'])
```
- 메인 페이지를 반환하는 GET / 엔드포인트가 추가되었다. 'username'이라는 키가 session에 이미 존재하는지 체크해서, 로그인 여부를 검사한다. 로그인이 안 되어있다면 get_login()에 해당하는 경로로 리다이렉트한다. 로그인이 이미 되어있다면 현재 로그인된 이용자의 유저네임인 session\['username']을 index.html에 전달해서 렌더링하여 반환한다.

### 게시판 기능 구현하기
- **CRUD**: 생성, 조회, 수정, 삭제의 약자로, 데이터를 다루는 네 가지 기본적인 작업을 뜻한다.
- **GET /posts**: 게시글 목록을 보여주는 페이지 반환
- **GET /posts/new**: 게시글을 작성할 수 있는 페이지 반환
- **POST /posts/new**: 게시글 작성을 실제로 처리
- **GET /posts/\<post_id>**:post_id에 해당하는 게시글을 보여주는 페이지를 반환. 게시글 내용을 보여주는 페이지이며 게시글을 수정하러 갈 수 있는 버튼과 삭제할 수 있는 버튼을 보여줌
- **GET /posts/\<post_id>/edit**: post_id에 해당하는 게시글 수정을 실제로 처리.
- **GET /posts/\<post_id>/delete**: 삭제할 수 있는 페이지 반환
- **POST /posts/\<post_id>/delete**: 실제 삭제 처리


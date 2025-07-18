### condition
해당 구문을 사용할 때에는 두 가지 필수 조건을 만족해야 한다. 첫 번째로는 이전 SELECT 구문과 UNION을 사용한 구문의 실행 결과 중 컬럼의 갯수가 같아야 한다. 

### application logic
SQL injection은 애플리케이션 내부에서 사용하는 데이터베이스의 데이터를 조작하는 기법이다. 만약, 특정 쿼리를 실행했을 때 쿼리의 실행 결과가 애플리케이션에서 보여지지 않는다면 공격자 입장에서는 데이터베이스의 정보를 추측하기 어렵다. 대부분의 애플리케이션은 데이터베이스의 결과에 따라 각기 다른 기능을 수행한다. 예를 들어, 특정 번호의 게시물을 조회할 때 번호에 일치하는 게시물이 있다면 이용자에게 보여주고, 없다면 게시물이 존재하지 않는다는 메시지를 출력한다. 이를 통해 공격자는 참과 거짓을 구분하여 공격을 수행할 수 있다. 
아래는 flask 프레임워크를 사용한 예제 코드이다. 코드를 살펴보면, 인자로 전달된 username을 쿼리로 사용하면서 SQL injection이 발생하며, 해당 쿼리의 결과로 username 이 admin일 경우 참을 아닐 경우 거짓을 반환한다.
```python
## pip3 install PyMySQL //pymysql 라이브러리를 설치하기 위한 명령어
from flask import Flask, request
import pymysql
app=Flask(__name__)
def getConnection():
	return pymysql.connect(host='localhost',user='dream',password='hack',db='dreamhack',charset='utf8')
@app.route('/',methods=['GET'])
def index():
	username=request.args.get('username')
	sql="select username from users where username='%s'"%username
	conn=getConnection()
	curs=conn.cursor(pymysql.cursor.DictCursor)
	curs.execute(sql)
	rows=curs.fetchall()
	conn.close()
	if(rows[0]['username']=="admin"):
		return "True"
	else:
		return "False"
app.run(host='0.0.0.0',port=8000)
```

### logic 이용 공격
데이터베이스가 아래와 같이 구성되어 있다고 가정
- admin-Password_for_admin
- guest-guest_Password
UNION 절을 사용하면 두 개의 SELECT 구문의 결과를 반환하므로 참을 반환 가능. 아래는 SQL injection으로 새로운 쿼리를 삽입한 결과이다. 공격 코드를 삽입하면 UNION 절에서 admin을 반환하기 때문에 애플리케이션에서 참을 반환
>\/?username=' union select 'admin' -- -
>==> True

애플리케이션에서 admin을 반환해 관리자 권한으로 로그인하는 방법도 있지만 관리자 계정의 비밀번호를 알아내는 방법 또한 존재한다. SQL에서는 IF문을 사용해 비교구문을 만들 수 있다. 해당 구문을 통해 관리자 계정의 비밀번호를 한 글짜씩 알아낼 수 있다. 
>\/?username=' union select if(substr(password,1,1))='B', 'admin', 'not admin') from users where username='admin' -- -

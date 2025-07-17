## SQL injection
- SQL은 DBMS에 데이터를 질의하는 언어이다. 웹 서비스는 이용자의 입력을 SQL 구문에 포함해 요청하는 경우가 있다.
- 예를 들어, 로그인 시에 ID/PW를 포함하거나, 게시글의 제목과 내용을 SQL 구문에 포함한다. 
- SQL injection이 발생하면 조작된 쿼리로 인증을 우회하거나, 데이터베이스의 정보를 유출 가능
> SELECT \* FROM accounts WHERE uer_id='admin'
- admin 계정의 비번을 비교하지 않고 해당 계정의 정보를 반환하기 때문에 이용자는 admin 계정으로 로그인할 수 있다.

### blind SQL injection
- 앞서 SQL injection을 통해 의도하지 않은 결과를 반환해 인증을 우회하는 실습을 했다. 해당 공격은 인증 우회 이외에도 데이터베이스의 데이터를 알아낼 수 있다. 
- 이때 사용가능한 공격 기법은 blind SQL injection이다. 해당 공격 기법은 스무고개 게임과 유사한 방식으로 데이터를 알아낼 수 있다. 
- 이처럼 질의 결과를 이용자가 화면에서 직접 확인하지 못할 때 참/거짓 반환 결과로 데이터를 획득하는 공격 기법을 blind SQL injection이라고 한다.
- **substr**: 해당 함수는 문자열에서 지정한 위치부터 길이까지의 값을 가져온다.
> substr(string, position, length)
> substr('ABCD',1,1)='A'
> substr('ABCD',2,2)='BC'
- 두번째 조건을 보면 upw의 첫 번째 값을 아스키 형태로 변환한 값이 'a' 또는 'b'인지 질의
- 질의 결과는 로그인 성공 여부로 참/거짓 판단 가능
- 이처럼 쿼리문의 반환 결과를 통해 admin 계정의 비밀번호를 획득 가능

### blind SQL injection 공격 스크립트
파이썬은 HTTP 통신을 위한 다양한 모듈이 존재하는데, 대표적으로 requests 모듈이 있다. 해당 모듈은 다양한 메소드를 사용해 HTTP 요청을 보낼 수 있으며 응답 또한 확인할 수 있다. 

### 비관계형 데이터베이스
RDBMS는 SQL을 사용해 데이터를 조회 및 추가 그리고 삭제할 수 있다. NoSQL은 SQL를 사용하지 않고 복잡하지 않은 데이터를 저장해 단순 검색 및 추가 검색 작업을 위해 매우 최적화된 저장 공간인 것이 큰 특징이자 RDBMS와의 차이점이다. 이 외에도 키-값을 사용해 데이터를 저장하는 차이점이 존재한다.
RDBMS는 SQL이라는 정해진 문법을 통해 저장하기 때문에 한 가지의 언어로 다양한 DBMS을 사용할 수 있다. 반면에 NOSQL은 다양한 DBMS가 존재하기 때문에 각각 구조와 사용문법을 익혀야한다는 단점이 있다.

### MongoDB
JSON 형태인 도큐먼트를 저장하며, 다음과 같은 특징을 갖고 있다.
1. 스키마를 따로 저장하지 않아 각 컬렉션에 대한 정의가 필요하지 않다.
2. JSON 형식으로 쿼리를 작성할 수 있다.
3. _ id 필드가 primary key 역할을 한다.

- status가 A이고 qty가 30보다 작은 데이터를 조회
	- RDBMS:
		> SELECT * FROM inventory WHERE status = "A" and qty < 30;
		
	- MongoDB
		>db.inventory.find(
			{ $and: [
			    { status: "A" },
			    { qty: { $lt: 30 } }
		  ]}
		)

- 연산자
	- $eq: 같은값
	- $in: 배열 안의 값들과 일치하는 값
	- $ne: 지정된 값과 같지 않은 값
	- $nin: 배열 안의 값들과 일치하지 않는 값

- ex
>db.account.find()

>db.account.find(
>  { user_id: "admin" }
>)

>db.account.find(
>  { user_id: "admin" },
>  { user_idx:1, _ id:0 }
>)

### Redis
키-값의 쌍을 가진 데이터를 저장한다. 제일 큰 특징은 다른 데이터베이스와 다르게 메모리 기반의 DBMS이다. 메모리를 사용해 데이터를 저장하고 접근하기 때문에 읽고 쓰는 작업을 다른 DBMS보다 훨씬 빠르게 수행. 따라서 다양한 서비스에서 임시 데이터를 캐싱하는 용도로 주로 사용

### CounchDB
MongoDB와 같이 JSON 형태인 도큐먼트를 저장한다. 이는 웹 기반의 DBMS로 REST API 형식으로 요청을 처리한다. 

### NOSQL injection
nosql은 사용하는 DBMS에 따라 요청 방식과 구조가 다르기 때문에 각각의 DBMS에 대해 이해하고 있어야 한다. 
nosql injection은 sql injection과 공격 목적 및 방법이 매우 유사하다. 두 공격 모두 이용자의 입력값이 쿼리에 포함되면서 발생하는 문제이다. 입력값에 대한 타입 검증이 불충분할 때 발생.
mongodb는 sql 저장 데이터 자료형 외에도 오브젝트, 배열 타입을 사용할 수 있다. 오브젝트 타입의 입력값을 처리할 때에는 쿼리 연산자를 사용할 수 있는데 이를 통해 다양한 행위가 가능해진다. 

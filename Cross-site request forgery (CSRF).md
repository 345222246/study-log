## CSRF
- 웹 서비스는 쿠키 또는 세션을 사용해서 이용자를 식별한다. 임의 이용자의 쿠키를 사용할 수 있다면, 이는 곧 임의 이용자의 권한으로 웹 서비스의 기능을 사용할 수 있다. CSRF는 임의 이용자의 권한으로 임의 주소에 HTTP 요청을 보낼 수 있는 취약점이다. 공격자는 임의 이용자의 권한으로 서비스 기능을 사용해 이득을 취할 수 있다. 
- 예를 들어, 이용자의 계정으로 임의 금액을 송금해 금전적인 이득을 취하거나 비밀번호를 변경해 계정을 탈취하고, 관리자 계정을 공격해 공지사항 작성 등으로 혼란 야기할 수 있다.
- 아래 코드는 송금 기능을 수행하는 엔드포인트 코드 예시로, CSRF 취약점이 존재한다. 코드를 살펴보면, 이용자로부터 예금주와 금액을 입력받고 송금을 수행하낟. 이때 계좌 비밀번호, OTP 등을 사용하지 않기 때문에 로그인한 이용자는 추가 인증 정보 없이 해당 긴으 이용 가능.
```python
# 이용자가 /sendmoney에 접속했을때 아래와 같은 송금 기능을 웹 서비스가 실행함.
@app.route('/sendmoney')
def sendmoney(name):
	# 송금을 받는 사람과 금액을 입력받음.
	to_user=request.args.get('to')
	amount=int(request.args.get('amount'))

	# 송금 기능 실행 후, 결과 반환
	success_status=send_money(to_user, amount)

	# 송금이 성공했을 때,
	if sucess_status:
		# 성공 메시지 출력
			return "send success"
	# 실패
	else:
		return "send fail"
```

### CSRF 동작
- CSRF 공격에 성공하기 위해서는 공격자가 작성한 악성 스크립트를 이용자가 실행해야함. 이는 공격자가 이용자에게 메일을 보내거나 게시판에 글을 작성해 이용자가 이를 조회하도록 유도하는 방법이 있다. 
- CSRF 공격 스크립트는 HTML 또는 javascript를 통해 작성 가능
- 아래 코드는 ima 태그를 사용한 스크립트 예시. 이를 활용하면 이용자에게 들키지 않고 임의 페이지 요청 보낼 수 있음
>\<img src='http://bank.dreamhack.io/sendmoney?to=Dreamhack&amount=1337' width=0px height=0px>

- 아래 코드는 자바스크립트로 작성된 스크립트 예시. 새로운 창을 띄우고, 현재 창의 주소를 옮기는 등의 행위 가능
>/* 새 창 띄우기 */
> window.open('http://bank.dreamhack.io/sendmoney?to=Dreamhack&amount=1337');
> /* 현재 창 주소 옮기기 */
> location.href = 'http://bank.dreamhack.io/sendmoney?to=Dreamhack&amount=1337';
> location.replace('http://bank.dreamhack.io/sendmoney?to=Dreamhack&amount=1337');

### XSS와 CSRF 차이
- 공통점: 모두 클라이언트를 대상으로 하는 공격이며, 이용자가 악성 스크립트가 포함된 페이지에 접속하도록 유도해야함
- 차이점: 
	- XSS는 인증 정보인 세션 및 쿠키 탈취를 목적으로 하는 공격, 공격할 사이트의 오리진 스크립트를 실행
	- CSRF는 이용자가 임의 페이지에 HTTP 요청을 보내는 것을 목적으로 함. 또한, 공격자는 악성 스크립트가 포함된 페이지에 접근한 이용자의 권한으로 웹 서비스의 임의 기능 실행.

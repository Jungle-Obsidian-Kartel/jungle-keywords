
- 유저 프로그램들은 ==자신들이 컴퓨터 전체를 소유한다는 환상== 하에 쓰여진다.
	- 따라서 각각의 프로세스들을 로드하고 실행할 때, 메모리, 스케쥴링, 그 외 다른 상태들이 프로세스가 가진 이 환상을 만족시키는 방향으로 관리한다.

### 가상 메모리

핀토스의 가상 메모리는 2개의 영역으로 구분된다.

1. 유저 가상 메모리
	- 가상 주소 0 ~ `KERN_BASE`
2. 커널 가상 메모리
	- 나머지

하나의 프로세스 - 하나의 유저 가상 메모리

### 프로세스 문맥 교환

프로세스 문맥 교환이 일어날 때, 커널은 프로세서의 페이지 디렉토리 베이스 레지스터(PDBR)를 바꿈으로써 유저 가상 주소 공간 또한 전환한다.
스레드 구조체는 하나의 프로세스의 페이지 테이블을 가리키는 포인터를 가지고 있다.

### 커널 가상 메모리

- 커널 가상 메모리 : global
- 커널 가상 메모리는 어떠한 유저 프로세스 또는 커널 스레드가 CPU 제어권을 획득해 running인지에 관계 없이 항상 같은 방식으로 매핑한다.
- 핀토스에서 `KERN_BASE`에서 시작하는 커널 가상 메모리는 물리 메모리와 일대일 매핑된다.
- 다시 말해, 가상 주소인 `KERN_BASE`는 물리 주소 0에 매핑, 가상 주소 `KERN_BASE + 0x1234`는 물리 주소 0x1234에 매핑된다.
- 이 매핑 방식은 머신의 물리 메모리 사이즈까지 유효하다.

### 페이지 폴트

- 유저 프로그램은 자신의 유저 가상 메모리에만 접근 가능하다.
- 유저 프로그램이 커널 가상 메모리에 접근하려는 시도는 ==page fault==를 야기하고 프로세스는 종료된다.
- page fault는 `userprog/exception.c`에 있는 `page_fault()`라는 함수에 의해 이루어진다.
- 커널 스레드들은 커널 가상 메모리에 접근 가능하다.
- 만일 유저 프로세스가 running 상태라면, 이 유저 프로세스의 유저 가상 메모리에도 접근 가능하다.
- 하지만, 심지어 커널에서도 매핑되지 않은 유저 가상 주소로 메모리에 접근하려는 시도조차도 page fault를 야기한다.

- 시스템 콜의 일부로서, 커널은 유저 프로그램이 제공한 포인터들을 통해 메모리에 자주 접근해야 한다.
- 유저가 null 포인터나 매핑되지 않은 가상 메모리를 가리키는 포인터, 혹은 커널 가상 메모리 공간(KERN_BASE 위에 있는 공간)을 가리키는 포인터를 전달할 수 있기 때문에 커널은 매우 신중해야 한다.
- 잘못된 포인터들은 커널과 다른 running 프로세스들에게 아무 악영향을 미치지 않고 거부되어져야 하고, 이를 위해서 문제가 되는 프로세스를 종료시키고 그 프로세스의 자원들을 해제해야 한다.

### 유저가 전달한 잘못된 포인터에 대응하는 2가지 방법

1. 유저가 전달한 포인터에 문제가 없는지 검사한 후에 역참조
	- `thread/mmu.c` 와 `include/threads/vaddr.h`
	
2. 유저가 전달한 포인터가 KERN_BASE보다 아래 부분을 가리키고 있는지 검사한 후에 역참조
	- 잘못된 유저 포인터는 `userprog/exception.c`의 `page_fault()`함수 코드를 수정함으로써 당신이 page fault 처리 가능
	- 이 방법은 프로세서의 MMU를 활용하기 때문에 보통은 더 빠르고, 리눅스를 포함한 실제 커널에서도 많이 사용된다.

```c
/* Reads a byte at user virtual address UADDR.
 * UADDR must be below KERN_BASE. 
 * Returns the byte value if successful, -1 if a segfault *
 * occurred. */ 
static int64_t 
 get_user (const uint8_t *uaddr) {
	int64_t result;
	__asm __volatile (
	"movabsq $done_get, %0\n"
	"movzbq %1, %0\n"
	"done_get:\n" 
	: "=&a" (result) : "m" (*uaddr)); 
	return result; 
} 

/* Writes BYTE to user address UDST. 
 * UDST must be below KERN_BASE. 
 * Returns true if successful, false if a segfault occurred. */ 
static bool put_user (uint8_t *udst, uint8_t byte) { 
	int64_t error_code; 
	__asm __volatile ( 
	"movabsq $done_put, %0\n" 
	"movb %b2, %1\n" 
	"done_put:\n" 
	: "=&a" (error_code), "=m" (*udst) : "q" (byte)); 
	return error_code != -1; 
}
```

- 이 함수들 각각은 유저 주소가 KERN_BASE 아래에 존재한다는 게 검증되었다고 가정 
- 또한 이 함수들은 여러분이 `page_fault()` 를 수정했고, 그 결과로 커널의 page fault는 단지 `rax`를 -1로 설정하고, 이의 이전 값을 `%rip`로 복사한다고 가정

두 가지 방법 모두 메모리 누수 조심

### 메모리 레이아웃

```
USER_STACK +----------------------------------+
           |             user stack           |
           |                 |                |
           |                 |                |
           |                 V                |
           |           grows downward         |
           |                                  |
           |                                  |
           |                                  |
           |                                  |
           |           grows upward           |
           |                 ^                |
           |                 |                |
           |                 |                |
           +----------------------------------+
           | uninitialized data segment (BSS) |
           +----------------------------------+
           |     initialized data segment     |
           +----------------------------------+
           |            code segment          |
 0x400000  +----------------------------------+
           |                                  |
           |                                  |
           |                                  |
           |                                  |
           |                                  |
       0   +----------------------------------+

```

한 실행파일의 레이아웃을 보기 위해서는 `-p` 옵션과 함께 `objdump` 명령어를 실행







### 문자열을 저장하는 것

아래 코드에서 문자열을 스택에 저장합니다

```c
for (int i = argc - 1; i >= 0; i--) {
    size_t len = strlen(argv[i]) + 1;  // '\0' 포함
    if_->rsp -= len;
    argv_addr[i] = if_->rsp;  // 문자열이 저장된 위치의 주소
    memcpy(if_->rsp, argv[i], len);  // 문자열을 스택에 복사
}
```

이 부분은 각 문자열을 스택에 저장하는 과정입니다. 이 때, `argv_addr[i]`에는 문자열이 저장된 위치의 주소가 저장됩니다.

### 문자열의 주소를 저장하는 것

문자열의 주소를 저장하는 것은 `argv` 배열을 구성하는 과정에서 필요합니다. 이 주소들은 나중에 `argv` 배열의 각 요소가 문자열을 가리키도록 합니다:

```c
if_->rsp -= sizeof(char *);
*(char **)(if_->rsp) = 0;  // NULL sentinel

for (int i = argc - 1; i >= 0; i--) {
    if_->rsp -= sizeof(char *);
    memcpy(if_->rsp, &argv_addr[i], sizeof(char *));  // 문자열의 주소를 스택에 복사
}
```

여기서 `argv_addr[i]`에는 이전 단계에서 스택에 저장된 각 문자열의 주소가 들어 있습니다. 이 주소들을 다시 스택에 복사하여 `argv` 배열을 구성하는 것입니다. 이 과정이 필요한 이유는, `argv` 배열이 각 문자열을 가리키는 포인터들의 **배열** 이어야 하기 때문입니다.

### 요약

- **문자열 저장**: 실제 문자열 데이터를 스택에 복사하고, 그 주소를 `argv_addr` 배열에 저장합니다.
- **주소 저장**: `argv_addr` 배열에 저장된 주소들을 다시 스택에 복사하여 `argv` 배열을 구성합니다. 이 `argv` 배열은 각 문자열의 시작 주소를 가리키는 포인터들의 배열입니다.

### 👨🏻‍🎨 그림으로 이해하기 🐭


#### 예제 명령: `/bin/ls -l foo bar`

#### 단계 1: 문자열을 스택에 저장
```
| Address      | Data        | 설명                  |
| ------------ | ----------- | --------------------- |
| 0x4747ffe4   | 'bar\0'     | `argv[3]`             |
| 0x4747ffe0   | 'foo\0'     | `argv[2]`             |
| 0x4747ffdd   | '-l\0'      | `argv[1]`             |
| 0x4747ffd5   | '/bin/ls\0' | `argv[0]`             |
```

``` c
for(int i = argc-1; i >= 0; i--) {

size_t len = strlen(argv[i]) + 1; // Null 종단자 포함해야함. 따라서 +1

distance += len;

argv_addr[i] = (if_->rsp - distance); // addr 배열에 저장

memcpy((if_->rsp - distance), argv[i], len); // rsp에 매개변수들을 넣어줌.

}
```

#### 단계 2: 8바이트 정렬
```
| Address      | Data        | 설명                  |
| ------------ | ----------- | --------------------- |
| 0x4747ffd0   | 0           | Word-align padding    |
```

``` c
while ( ((if_->rsp - distance) % 8) != 0 ) { // 8배수 패딩
distance++;
memset(if_->rsp - distance, 0, sizeof(uint8_t)); 
}
```

#### 단계 3: `argv` 배열을 스택에 저장
```
| Address      | Data        | 설명                  |
| ------------ | ----------- | --------------------- |
| 0x4747ffcc   | 0x4747ffe4  | `argv[3]`의 주소      |
| 0x4747ffc8   | 0x4747ffe0  | `argv[2]`의 주소      |
| 0x4747ffc4   | 0x4747ffdd  | `argv[1]`의 주소      |
| 0x4747ffc0   | 0x4747ffd5  | `argv[0]`의 주소      |
| 0x4747ffbc   | 0           | `argv[4] = NULL`      |
```

``` c
for (int i=argc; i>=0; i--) {
	distance+=8;
if (i == argc) {
	memset(if_->rsp - distance,0, sizeof( uintptr_t) );
}
else {
	memcpy(if_->rsp - distance , &argv_addr[i], sizeof( uintptr_t));
	}
}
```

#### 단계 4: 가짜 반환 주소를 추가
```
| Address      | Data        | 설명                  |
| ------------ | ----------- | --------------------- |
| 0x4747ffb8   | 0           | 가짜 반환 주소         |
```
``` c
/* 4 번 단계 */

if_->rsp -= distance;
if_->R.rsi = if_->rsp;
if_->R.rdi = argc; // 그냥상수

/* 5번 단계 */
if_->rsp -= sizeof( uintptr_t);
memset(if_->rsp, 0, sizeof( uintptr_t));
```
#### 최종 스택 상태
```
| Address      | Name        | Data                  | Type          |
| ------------ | ----------- | --------------------- | ------------- |
| 0x4747ffe4   | argv[3][...]| 'bar\0'               | char[4]       |
| 0x4747ffe0   | argv[2][...]| 'foo\0'               | char[4]       |
| 0x4747ffdd   | argv[1][...]| '-l\0'                | char[3]       |
| 0x4747ffd5   | argv[0][...]| '/bin/ls\0'           | char[8]       |
| 0x4747ffd0   | word-align  | 0                     | uint8_t[]     |
| 0x4747ffbc   | argv[4]     | 0                     | char *        |
| 0x4747ffcc   | argv[3]     | 0x4747ffe4            | char *        |
| 0x4747ffc8   | argv[2]     | 0x4747ffe0            | char *        |
| 0x4747ffc4   | argv[1]     | 0x4747ffdd            | char *        |
| 0x4747ffc0   | argv[0]     | 0x4747ffd5            | char *        |
| 0x4747ffb8   | return addr | 0                     | void (*) ()   |
```

## 핵심 포인트 🛎️

- 포인터를 사용하면 `&argv[i]`는 실제 문자열이 저장된 **메모리 주소**를 참조.
- 주소 자체를 사용하면 `argv[i]`는 문자열의 **첫 번째 바이트**를 참조합니다. 즉, 주소로 저장된 값을 올바르게 참조하지 못하고 단순히 값으로 인식하게 됩니다! 

따라서, `memcpy(if_->rsp, &argv_addr[i], sizeof(char *));` 코드가 각 문자열의 주소를 `argv` 배열에 저장하는 이유는 명령 줄 인자를 참조하기 위해서입니다. `argv_addr` 배열은 각 문자열이 저장된 주소를 담고 있고, 이를 통해 `argv` 배열을 구성하는 것입니다.
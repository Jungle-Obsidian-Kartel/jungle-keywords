```c
int process_exec (void *f_name)
```

### 수정해야 하는 부분?
---
1. `f_name`을 통째로 `file_name`으로 바꿔주고 있다.
    
    - `strtok_r` 를 사용해 토큰별로 쪼개준 다음 (공백 여러개 있을 때도 처리해야 함)
    
    ```
    // 최종 스택의 모습
    | Address     | Data           | Comment               |
    |-------------|----------------|-----------------------|
    | 0x4747fffc  | 'bar\\0'        | String "bar"          |
    | 0x4747fff8  | 'foo\\0'        | String "foo"          |
    | 0x4747fff5  | '-l\\0'         | String "-l"           |
    | 0x4747ffed  | '/bin/ls\\0'    | String "/bin/ls"      |
    | 0x4747ffe8  | 0              | Word alignment        |
    | 0x4747ffe0  | 0              | argv[4] = NULL        |
    | 0x4747ffd8  | 0x4747fffc     | argv[3] = "bar\\0"     |
    | 0x4747ffd0  | 0x4747fff8     | argv[2] = "foo\\0"     |
    | 0x4747ffc8  | 0x4747fff5     | argv[1] = "-l\\0"      |
    | 0x4747ffc0  | 0x4747ffed     | argv[0] = "/bin/ls\\0" |
    | 0x4747ffb8  | 0              | Fake return address   |
    ```
    
    - 위의 스택과 같이 메모리에 할당해준다.
    
2. `load`에서 `file_name`에 해당하는 바이너리 파일을 로드하고 있으므로, 사용자로부터 입력받은 매개변수에서 실제로 `file_name`에 해당하는 부분만 `load`의 인자로 전달해주어야 한다.
    
3. `load`의 `set_up_stack`에서 스택 공간에 대한 할당(`palloc_get_page`)을 하고 있으므로 이후에 매개변수를 스택에 할당해주어야 한다.


### 디버깅
- 실제로 스택에 들어갔는지 확인을 위해 [[hex_dump]] 를 사용한다. 
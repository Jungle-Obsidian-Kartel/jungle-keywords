- 사용 예제
    ```c
    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>
    
    char *strtok_r(char *s, const char *delimiters, char **save_ptr) {
        char *token;
    
        if (s == NULL)
            s = *save_ptr;
    
        while (strchr(delimiters, *s) != NULL) {
            if (*s == '\\0') {
                *save_ptr = s;
                return NULL;
            }
            s++;
        }
    
        token = s;
        while (strchr(delimiters, *s) == NULL)
            s++;
        if (*s != '\\0') {
            *s = '\\0';
            *save_ptr = s + 1;
        } else
            *save_ptr = s;
    
        return token;
    }
    
    int main(int argc, const char * argv[]) {
        
        char s[] = "  String to  tokenize. ";
        char *token, *save_ptr;
    
    //    for (token = strtok_r(s, " ", &save_ptr); token != NULL;
    //         token = strtok_r(NULL, " ", &save_ptr))
    //        printf("'%s'\\n", token);
    
        token = strtok_r(s, " ", &save_ptr);
        printf("'%s'\\n", token);
        
        return 0;
    }
    
    ```

> [!info]
> [[jungle-keywords/PintOS/userprog/argument passing/process_exec|process_exec]] 에서 받을 매개변수를 파싱하기 위해 사용하는 함수이다.

### strchr (문자열 찾기)

```c
char *strchr(const char *str, int c);
```

- 문자열 `str` 내에 `c` 문자가 있는지 확인
- `c`는 `int` 타입이지만 문자로 취급한다.
- 반환되는 `ptr`은 찾은 문자의 포인터이므로, 출력한다면 그 포인터의 뒷 부분이 전부 출력된다.
    - strchr(”Hello, World!”, ‘o’) → “o, World!” 출력


```c
char *strtok_r(char *s, const char *delimiters, char **save_ptr) {
    char *token;

    if (s == NULL)
        s = *save_ptr;

		// 1.
    while (strchr(delimiters, *s) != NULL) {
        if (*s == '\\0') {
            *save_ptr = s;
            return NULL;
        }
        s++;
    }

		// 2.
    token = s;
    while (strchr(delimiters, *s) == NULL)
        s++;
        
    if (*s != '\\0') {
        *s = '\\0';
        *save_ptr = s + 1;
    } else
        *save_ptr = s;

    return token;
}
```

### 1. `while (strchr(delimiters, *s) != NULL)`

> 주어진 문자열(`s`)의 시작부터 검사하여 `delimiters`가 나올 때까지 s++를 수행한다.

> [!example]
> **ex. “ String to tokenize. “,**
> **delimiter = “ “인 경우**
> 
> - “ “ x3 pass
> - ‘S’에서 while 종료
> 
> ⇒ 결과: “String to tokenize. “

### 2. `while (strchr(delimiters, *s) == NULL)`

> `s`에서부터 다음 `delimiters`가 나올 때까지 s++를 수행한다.

> [!example]
> **ex. “String to tokenize. “**
> **delimiter = “ “인 경우**
> 
> - ‘S’, ‘t’, ‘r’, ‘i’, ‘n’, ‘g’ pass
> - “ “에서 while 종료
> 
> ⇒ 결과: “String”

### 반환되는 값
---
- `token`에 2번 while을 수행하기 전 s의 위치가 저장된다.
- 이후의 if문에서 delimiters를 만나면 `\0`로 바꿔주었기 때문에, 첫 단어가 반환된다.
- 즉, “String\\0to tokenize. “이다.
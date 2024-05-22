```c
int process_exec (void *f_name)
```

### 수정해야 하는 부분?
---
1. `f_name`을 통째로 `file_name`으로 바꿔주고 있다.
	- [[strtok_r]] 를 사용해 토큰별로 쪼개준 다음 (공백 여러개 있을 때도 처리해야 함)
	- `intr_frame` 구조체 변수 `_if`의 `gp_registers`에 존재하는 argument register에 차례대로 매개변수를 저장하면 될 것 같다?

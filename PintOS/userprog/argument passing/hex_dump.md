- 사용 예시

```c
hex_dump(_if.rsp, _if.rsp, USER_STACK - _if.rsp, true);	
```

> [!info]
>  버퍼에 존재하는 내용을 읽어주는 함수 
>  (`stdio.c`에 존재)


- argument passing이 제대로 되었는지 디버깅을 위해 사용


![[Pasted image 20240523142630.png]]

- 이렇게 출력되어야 한다.

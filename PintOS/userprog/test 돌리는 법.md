---
sticker: emoji//1f608
---
# 전체 테스트

> `userprog` 파일에서 make check

아무것도 진행하지 않은 상태에서는 77개의 test가 성공해야 합니다.

![](https://i.imgur.com/zqomE8C.png)


>[!tips]
***만약 test가 진행되지 않는다면 ...?!***
- `chmod -R 777 <폴더 경로>` 명령어를 실행해 하위 디렉토리에 수정 권한 주기
- `user prog` 파일에서 `pintos-mkdisk filesys.dsk 10` 명령어 실행 후 make check!

# 단일 테스트

***1. args-single onearg test***

- stack 토근이 잘 적제되었는지 확인하고 싶으면 hex_dump() 함수를 선언해주고 해당 단일 테스트를 실행해주시면 됩니다.
```
pintos --fs-disk=10 -p tests/userprog/args-single:args-single -- -q -f run 'args-single onearg'
```

> ex. `/bin/ls -l foo bar`

1️⃣ **명령어를 공백 단위로 자른다.** 
- ex. `/bin/ls`, `-l`, `foo`, `bar`

2️⃣ **스택 위에 문자열을 추가한다.**
- 순서는 상관 없다.

3️⃣ **주소의 끝을 표시하는 null sentinel을 추가한다.**
- `argv[argc]`가 null pointer가 되도록 매개변수의 주소를 뒤에서부터 차례대로 스택에 추가한다.

4️⃣ `**%rsi`가 `argv[0]`의 주소를 가리키도록 하고,** 
	**`%rdi`를 `argc`로 설정해준다.**

5️⃣ **가짜 리턴 주소(0)를 푸시한다.**

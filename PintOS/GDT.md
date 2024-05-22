- GDT를 풀어쓰자면 글로벌 디스크립터 테이블(Global Descriptor Table, GDT)이며, 말 그대로 무언가에 대한 기술서를 테이블 형식으로 한데 모아둔 형태를 의미합니다.

- 여기에서 무언가는 각 [[segment]] 영역을 나타냅니다. 즉, segment 영역에 대한 데이터를 일정한 [[descriptor]] 형식으로 기술하고 이를 하나의 테이블에 모아두고자 하는 것이 GDT를 사용하는 목적입니다.

![](https://i.imgur.com/ZakTTbP.png)

출처와 더 자세한 설명
https://itguava.tistory.com/14
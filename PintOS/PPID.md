- PPID 는 Parent Process ID 의 줄임말로 부모 [[process]]의 PID 를 의미한다.

- 부모 프로세스는 현재 실행 중인 프로세스 (자식 프로세스) 를 [[system call]]을 호출하여 생성한 프로세스를 의미한다.

- 만약 부모 프로세스가 자식 프로세스보다 일찍 종료되는 경우 자식 프로세스는 [[orphan process]]가 되어 PPID 로 init process 의 PID, 1을 가지게 된다.
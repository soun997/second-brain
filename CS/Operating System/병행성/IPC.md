
> 프로세스 간 통신, Interprocess Communication

협력 작업(공유/통신)을 하는 프로세스 사이에서는 정보의 교환이 필요하다.
이 때 사용하는 것이 바로 IPC 기법이다.

## IPC의 종류

>대부분 두 가지 방법을 혼용해서 사용한다.

### [[Shared Memory]]

속도는 빠르나, 충돌에 대한 관리가 매우 복잡하다.

### [[Message Passing]]

[[System Call]]을 통해 전달되기 때문에 속도는 느리지만, 충돌에 대한 문제가 없다.


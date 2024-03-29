
프로세스가 운영체제의 서비스를 요청하기 위한 방법

## System Call이 느린 이유

사용자 모드에서 커널 모드로의 전환을 포함하기 때문에, 함수 호출에 비해 많은 비용이 소모된다.

- **사용자 모드와 커널 모드의 전환:** 시스템 호출을 실행하려면 프로세스가 사용자 모드에서 커널 모드로 전환해야 합니다. 이 전환은 CPU 레지스터를 저장하고 복원해야 하며, 이는 시간이 소요되는 작업입니다.

- **데이터 전송:** 시스템 호출은 종종 운영 체제와 프로세스 간의 데이터 전송을 포함합니다. 이 데이터 전송은 메모리에서 메모리로의 전송이거나 메모리에서 주변 장치로의 전송일 수 있습니다. 메모리에서 메모리로의 전송은 상대적으로 빠르지만, 메모리에서 주변 장치로의 전송은 속도가 느릴 수 있습니다.

- **운영 체제의 처리:** 시스템 호출을 처리하기 위해 운영 체제는 다음과 같은 작업을 수행해야 합니다.
    - 시스템 호출의 유효성을 검사합니다.
    - 시스템 호출에 필요한 데이터를 수집합니다.
    - 시스템 호출을 수행합니다.
    - 시스템 호출의 결과를 프로세스에 반환합니다.


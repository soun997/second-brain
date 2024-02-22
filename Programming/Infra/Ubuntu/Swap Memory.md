
>실제 메모리 RAM은 가득 찼지만 더 많은 메모리가 필요할 때, 디스크 공간을 이용하여 부족한 메모리를 대체할 수 있는 공간

일종의 가상 메모리 개념이다.

실제 메모리가 아닌 하드디스크를 이용하는 것이기 때문에 메모리 속도는 떨어진다.
- 실제 운영환경에서는 스왑 메모리를 잘 사용하지 않는다고 한다.

EC2 메모리는 콩알만하니까 Swap Memory를 사용하려고 한다.
- Swap Memory는 일반적으로 서버 메모리의 2배로 설정한다고 한다.

## 설정 방법

```bash
free -h # 현재 메모리 영역 확
sudo fallocate -l 8G /swapfile # Swap 영역 할당 (일반적으로 서버 메모리의 2배)
sudo chmod 600 /swapfile # Swapfile 권한 수정
sudo mkswap /swapfile # Swapfile 생성
sudo swapon /swapfile # Swapfile 활성화
free -h # swap 영역이 할당 되었는지 확인
```

## Ref

[Swap Memory란? EC2 Swap memory설정법](https://jaykos96.tistory.com/13)
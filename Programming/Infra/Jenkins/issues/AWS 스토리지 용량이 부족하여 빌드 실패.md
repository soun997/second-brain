
## Jenkins가 멈췄다...

평소와 같이 PR을 올리고 Jenkins의 성공 알림을 기다리고 있었다.
그런데 한참이 지나도 디스코드에 알림이 오지 않는 것이다...!!

호다닥 Jenkins Dashboard에 접속하여 확인한 결과 Jenkins가 멈춰있었다...
- 오프라인이라고 뜬다.

확인한 결과 스토리지 용량이 부족해서 더 이상 Pipeline을 실행할 수 없다는 오류였다.


## 범인은 Docker?

`df -h` 명령어를 통해 스토리지를 확인하니 전체 용량인 30GB가 거의 꽉 차있었다.

누가 범인인걸까?

```sh
du -hsx * | sort -rh | head -n 10 # 디스크 사용량으로 정렬하여 확인
```

처음에는 Jenkins가 주범이라 생각하여 위의 명령어를 통해 Jenkins가 차지하는 용량을 확인하였다.

![[Pasted image 20231216045047.png]]

넌 아니구나...

두 번째로는 [[Docker]]를 의심했다.
생각해보니  [[Docker Build Cache]]를 사용하고 있었기 때문에 분명 이 친구가 문제일 것이라고 확신했다.

```bash
docker system prune -a
```

위 명령어를 통해 사용하지 않는 모든 컨테이너, 이미지를 삭제하고 빌드 캐시를 삭제하고자 했다.

![[Pasted image 20231216045309.png]]

현재 실행 중인 컨테이너에는 영향이 가지 않기 때문에 과감하게 실행했다.

오랜 시간이 흐른 후...

![[Pasted image 20231216045348.png]]

Docker 너... 정말 많이도 해먹었구나...


## 해결 방법

cron을 사용하여 주기적으로 빌드 캐시를 삭제해주는 것이 좋을 것 같다!


## Ref
[Jenkins - 디스크 공간 확보하기](https://velog.io/@cid-yoon/Jenkins-%EB%94%94%EC%8A%A4%ED%81%AC-%EA%B3%B5%EA%B0%84-%ED%99%95%EB%B3%B4%ED%95%98%EA%B8%B0)
[docker system purne](https://vhxpffltm.tistory.com/198)
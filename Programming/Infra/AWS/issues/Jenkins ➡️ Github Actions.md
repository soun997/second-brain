## 서론

[[EC2 다운그레이드]]
Swap 영역을 지정함으로써 t3.small에서 Jenkins로 인해 서버가 다운되는 것을 해결하였다.
또한 t3.medium에서 t3.small로 변경한 것이기 때문에 청구되는 요금 또한 절반으로 줄었다.

그러나 t3.small 또한 월 17$로 결코 저렴한 비용은 아니다.
게다가 t3.small을 사용하는 이유가 단지 Jenkins를 돌리기 위해서라면 Jenkins 외의 다른 대안을 찾는 것이 더 이득일 수도 있다.

이에 대한 대안으로 Jenkins를 **Github Action으로 마이그레이션**하고, 우리 서비스에 적당한 인스턴스가 무엇인지 다시 고민해보려고 한다.


## 또 다른 대안, Github Actions

먼저 제일 중요한 문제인 과금 정보를 확인해보았다.

![[Pasted image 20240310014922.png]]
ref) [GitHub Actions 요금 청구 정보](https://docs.github.com/ko/billing/managing-billing-for-github-actions/about-billing-for-github-actions)

무료라니..!


## Github Actions 적용기

[[Github Actions 적용기]]

많은 삽질 끝에, 기존의 Jenkins 파이프라인을 Github Actions으로 완전하게 옮길 수 있었다.


## 성과

AWS 청구 요금이 t3.small에 비해 **절반 가격**으로 줄었다!!

게다가 **빌드 속도까지 80% 정도 향상**되어 팀의 생산성에 큰 기여를 했다.

**서비스에 알맞는 아키텍처 선택하는 기준에 대하여 조금 갈피가 잡힌 것 같다.**
앞으로 사이드 프로젝트를 할 때는 가볍게 t3.micro 정도로 사용하면 될 것 같다!!


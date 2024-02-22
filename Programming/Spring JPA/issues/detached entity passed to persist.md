---
sticker: emoji//2705
---
Cascade 옵션을 가지고 있는 부모 엔티티의 자식 엔티티를 update하는 로직을 테스트 중에 해당 오류를 마주했다.

## 발생 이유

기존의 자식 엔티티 인스턴스와 다른 인스턴트이지만, DB에 이미 저장되어 있는 id 필드를 가지고 있는 경우에 발생한다.
- detached 되어있다고 인식하는 듯하다.

업데이트를 위해 dto에 id 필드를 넣은 채로 들어오고 자식 엔티티로 매핑될 것이지만
id 필드가 있는 상태이기 때문에 detached된 엔티티라고 인식하기 때문이다.
- 기대했던 동작은 persist가 아닌 merge가 호출되어 자식 엔티티의 정보가 업데이트 되는 것이다.

## 해결 방법

두 가지의 해결 방법이 있다.

### 수정하고자 하는 자식 엔티티를 먼저 찾는다.

수정 여부를 확인해주어야 한다는 단점이 있다.
성능적으로도 이점을 가지는 지는 확인이 필요할 것 같다.

### update가 아닌 delete&insert 전략을 취한다.

불필요한 delete와 insert 쿼리가 발생한다는 단점이 있다.
대신 매우 처리하기 간편해진다.

## 추가 과제

update 방식과 delete&insert 방식의 성능을 비교해보아도 좋을 것 같다.


## Ref

[Update VS Delete+Insert!!! 뭐가 더 나을까요?](https://www.sqler.com/board_SQLQA/925739)
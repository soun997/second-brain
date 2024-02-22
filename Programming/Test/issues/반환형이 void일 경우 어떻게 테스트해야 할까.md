
## 무엇을 검증해야 하는가?

데이터를 저장하거나 업데이트 하는 경우 메서드의 반환값이 없는 경우가 있다.

현재 진행하고 있는 프로젝트에서는 반환값은 없지만, 데이터를 저장하는 로직이 단순하지 않은 메서드가 있다.
이러한 메서드는 테스트가 필요하지 않을까?
필요하다면 어떻게 테스트 할 수 있을까?


## 행위를 검증하자

위와 같은 상황에서,

**Repository는** 데이터베이스에 데이터를 저장해야 하는 책임이 있다.
그렇기 때문에 제대로 저장이 되는지 검증하면 된다.

**Service는** Repository의 메서드 중 해당 비즈니스 로직을 처리하기에 가장 적절한 메서드를 호출해야 하는 책임이 있다.
우리는 Service의 메서드가 본인의 책임을 다 하는지 **행위를 검증**하면 된다!

### `verify()` 혹은 `then()`을 이용하여 행위 검증

프로젝트에서 구현한 데이터 저장 메서드는 다음과 같다.

```java
public void add(MealLogAddRequest mealLogAddRequest) {  
  
    MealLog mealLog =  
        mealLogRepository.save(mealLogMapper.mealLogAddRequestToEntity(mealLogAddRequest));  
    mealRepository.saveAll(  
            mealLogAddRequest.meals().stream()  
                    .map(meal ->  mealMapper.mealAddRequestToEntity(  
                                        meal,  
                                        mealLog,  
                                        mealDataQueryService.search(meal.mealDataId())))  
                    .toList());  
}
```

`MealLogAddRequest`에 대응되는 적절한 `MealLog` 엔티티가 있고, repository 메서드가 잘 작동한다는 전제 하에
과연 `add()` 메서드는 적절한 repository 메서드들을 호출할까?

이 때, Mockito 혹은 BDDMockito의 `verity()`나 `then()` 메서드를 사용하여 이를 검증할 수 있다.




## Ref

[반환형이 void인 메서드, 어떻게 테스트할까? (with 행위 검증)](https://kokodakadokok.tistory.com/entry/%EB%B0%98%ED%99%98%ED%98%95%EC%9D%B4-void%EC%9D%B8-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%85%8C%EC%8A%A4%ED%8A%B8%ED%95%A0%EA%B9%8C-with-%ED%96%89%EC%9C%84-%EA%B2%80%EC%A6%9D)
## 서론

```java
public record MealLogDetailsDto(  
        Long id,  
        MealType mealType,  
        List<MealDetailsDto> meals,  
        List<MealImage> mealImages,  
        Member member) {  
  
    public IntakeNutrients getTotalIntakeNutrients() {  

        return new IntakeNutrients(  
                calculateTotal(MealDetailsDto::calorie, meals),  
                calculateTotal(MealDetailsDto::carbs, meals),  
                calculateTotal(MealDetailsDto::protein, meals),  
                calculateTotal(MealDetailsDto::fat, meals));  
    }  
  
    private Integer calculateTotal(ToIntFunction<MealDetailsDto> func, List<MealDetailsDto> meals) {  
  
        return meals.stream().mapToInt(func).sum();  
    }  
}
```

기능 구현 후 리팩토링을 하기 위해 코드를 다시 들여다 봤는데
문득 **DTO에도 비즈니스 로직이 섞여있어도 되는가?**  하는 생각이 들었다.

우선 지금 개발 중인 애플리케이션은 다음과 같은 방식으로 요청에 대해 응답한다.
1. Service에서 Entity를 Dto로 변환
2. Controller에서 Dto를 Response로 변환



관계를 정리해보자.
Service는 Repository에게 `MealLog`엔티티를 가져오라고 메세지를 보낸다.

처음에 나는 Entity를 DTO에 매핑할 때 필요한 메서드라고 생각하여 저렇게 구현하였다.
그러나 만약 `getThumbnail`
- Presentation, Business Layer에 동시에

## DTO와 Domain을 분리하는 이유

### 관심사의 분리

OOP에서는 관심사(concern)의 분리를 통해 복잡한 시스템을 효율적으로 구축할 수 있다.
- 애플리케이션에 같은 영향을 미치는 코드들을 분리한다.

우리가 사용하는 Layered Architecture 또한 관심사를 분리한 것이라고 볼 수 있다.
- 관심사의 수평적 분리


[[DTO]]는 계층 간 ==데이터를 전달하는 역할==이다.
Domain은 Business Layer에서 ==본인의 비즈니스 로직을 수행하는 역할==이다.

(책 '오브젝트'에서는 역할이란 협력 안에서 책임을 다하는 후보(Candidate)라고 정의하였다)

만약 DTO와 Domain을 분리하지 않으면 해당 객체가 **너무 많은 책임**을 가지게 된다. (**God Class**)
- 책임은 해당 객체가 하는 것 + 아는 것으로 이루어져 있다.
이는 **관심사의 분리가 제대로 되지 않은 것**이고, 결국 **변경에 닫혀있는 코드**를 만들 수 밖에 없다.
- 변경의 여파가 클 경우, 변경이라는 행위 자체에 대해 소극적이게 된다.

## 결론



## Ref

[DTO는 왜, 언제 사용할까?](https://e-una.tistory.com/72)

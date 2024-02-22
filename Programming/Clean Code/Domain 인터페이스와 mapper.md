---
sticker: emoji//1f914
---
내가 Domain 인터페이스로 제공할 필요가 있다고 생각한 메서드를 팀원이 mapper 클래스로 빼는 것이 어떻겠냐는 피드백을 받았다.
- `MealLog`는 `MealImage`와 일대다 관계를 맺는다.
- `MealLog` 상세 조회 시, `MealImage`들의 `imageUrl` 필드를 리스트로 가져와야 한다.
- 그렇기 때문에 `MealLog`에 `imageUrl` 리스트를 반환하는 인터페이스를 생성하였다.

그러나 위의 로직이 `MealLog` mapper가 수행해야 하는 부분이라는 관점인 것이다.

----

## Mapping 로직

```java
// RecipeService.java
public void add(RecipeAddRequest request, Long memberId) {  
  
    Recipe recipe = recipeMapper.toEntity(request.name(), memberQueryService.search(memberId));  
    List<RecipeType> recipeTypes = request.recipeType().stream().map(recipeMapper::toRecipeType).toList();  
    List<RecipeImage> recipeImages = request.imageUrls().stream()  
            .map(recipeMapper::toRecipeImage).toList();  
    List<RecipeIngredient> ingredients = request.ingredients().stream()  
            .map(recipeMapper::toRecipeIngredient).toList();  
    List<RecipeDescription> descriptions = IntStream.range(1, request.descriptions().size())  
            .mapToObj(idx -> recipeMapper.toRecipeDescription(idx, request.descriptions().get(idx)))  
            .toList();  
    recipe.update(recipeTypes, recipeImages, ingredients, descriptions);  
}
```

정말 어지러운 코드다.
mapper가 수행해주어야 할 mapping 로직이 service에 그대로 노출되어 있다.
이것을 어떻게 해결할 수 있을까?

### 시도

mapping 로직을 모두 mapper로 옮겨주었다.

```java
// RecipeMapper.java
default Recipe toEntity(RecipeAddRequest request, Member member) {  
  
    Recipe recipe = Recipe.builder().name(request.name()).member(member).build();  
  
    List<RecipeType> recipeTypes =  
            request.recipeType().stream()  
                    .map(vegetarianType -> toRecipeType(vegetarianType, recipe))  
                    .toList();  
    List<RecipeImage> recipeImages =  
            request.imageUrls().stream()  
                    .map(imageUrl -> toRecipeImage(imageUrl, recipe))  
                    .toList();  
    List<RecipeIngredient> ingredients =  
            request.ingredients().stream()  
                    .map(ingredient -> toRecipeIngredient(ingredient, recipe))  
                    .toList();  
    List<RecipeDescription> descriptions =  
            IntStream.range(1, request.descriptions().size())  
                    .mapToObj(  
                            idx ->  
                                    this.toRecipeDescription(  
                                            idx, request.descriptions().get(idx), recipe))  
                    .toList();  
  
    recipe.update(recipeTypes, recipeImages, ingredients, descriptions);  
  
    return recipe;  
}
```

```java
public void add(RecipeAddRequest request, Long memberId) {  
  
    Recipe recipe = recipeMapper.toEntity(request, memberQueryService.search(memberId));  
    recipeRepository.save(recipe);  
}
```

**장점**
- mapper 코드는 어지러워졌지만 service 레이어의 로직은 비교적 간단해졌다.
	- 특히 드러날 필요가 없는 mapping 로직을 숨길 수 있었다.
- `Recipe` 등록과 관련된 dto가 수정되어도 mapper 클래스만을 수정하면 된다.

**단점**
- `Recipe` mapper가 자식 엔티티의 변환까지 전담한다.
	- mapper가 비대해졌다.
	- 일대다 관계에서 자식 엔티티를 부모 엔티티와 함께 다뤄줘야 하는지?에 대한 고민을 하게 되었다.
	- mapper가 다른 mapper를 의존하도록 하면 볼륨이 더 줄어들 순 있겠다. 그런데 그래도 되나..?

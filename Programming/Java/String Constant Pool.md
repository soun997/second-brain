## String Constant Pool이란?

Java에서 문자열 리터럴을 저장하는 독립된 영역이다.
- `String`은 **불변객체**이기 때문에 문자열의 생성 시, String Constant Pool에 저장된 리터럴을 재사용할 수 있다.
- String Constant Pool에 저장된 문자열은 **모두 같은 객체를 사용**하게 된다.
- 이는 디자인 패턴에 **Flyweight Pattern(플라이웨이트 패턴)**을 참고하면 이해가 더 쉬울 것이다.

String Constant Pool은 일반적으로 GC의 대상이 되지 않지만, 문자열 참조 대상이 없는 경우에는 선택적으로 GC 대상이 되기도 한다.


## 어떤 것이 String Constant Pool에 저장될까?

### 소스코드에 문자열을 선언한 경우

```java
String str1 = "Hello world!";  // String Constant Pool에 저장
String str2 = new String("Hello world!");  // Heap 영역에 저장

System.out.println(str1 == str2);  // false
System.out.println(str1.equals(str2))  // true
```

`new` 키워드를 사용하여 문자열을 생성하게 되면, String Constant Pool이 아닌 Heap 영역에 새로 저장하게 된다.
- 객체를 재사용하지 못하기 때문에, 성능&메모리 최적화를 위해 `new` 연산은 지양하는 것이 좋다.
- `String.valueOf()` 메서드도 내부적으로는 `new` 키워드를 사용하여 String을 생성한다.
	- 숫자는 변화되기 쉽기 때문에 그렇게 구현했나보다..! 상황에 맞게 사용!!

```java
String bananaMilk1 = banana + "Milk";   // Heap 영역에 저장  
String bananaMilk2 = "bananaMilk";  // String Constant Pool에 저장  
String bananaMilk3 = "bananaMilk";  // String Constant Pool에 있는 값을 가져옴  
System.out.println(bananaMilk1);  
System.out.println(bananaMilk2);  
System.out.println(bananaMilk1 == bananaMilk2); // false  
```

### `String.intern()` 메서드를 실행한 경우

```java
new String("Hello world!").intern();  // String Constant Pool에 강제로 저장
new String("My name is Soyun").intern(); // String Constant Pool에 강제로 저장
new String("Hello world!").intern();  // 이미 존재하기 때문에 저장되지 않음
```

`intern()` 메서드는 String Constant Pool에 해당 문자열이 있는지 검증하고 없다면 저장, 있다면 기존에 저장되어 있는 레퍼런스를 반환한다.

그러나 만약 **동적인 문자열에 `intern()` 메서드를 사용**한다면, 상수 풀이 사용하는 메모리가 점점 커지게 될 것이다. (String Constant Pool은 일반적으로 GC 대상이 되지 않는 것을 기억하자!)
- 그래서 `+` 연산을 사용하지 말라는 것인가..? ➡️ 아닌듯! https://dkswnkk.tistory.com/584

#### 주의
- 동적으로 생성되는 문자열은 상수 풀에 저장되지 않는다.
- ORM, MyBatis와 같은 DB에서 조회해 오는 String, File에서 읽어들이는 String
- 문자열을 비교해야 할 때는 `==` 보다는 `equals()`를 사용하자!


## Ref

https://deveric.tistory.com/123
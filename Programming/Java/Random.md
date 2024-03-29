Java에서 Random을 사용하기 위해 ==난수값==을 지정해주어야 한다.

왜 우리가 직접 난수값을 지정해주어야 할까?

그 이유는 컴퓨터가 직접 난수값을 생성할 수 없기 때문이다.
컴퓨터는 기본적으로 정해진 입력에 따라 정해진 출력을 만들어낸다.
- 이를 결정적 유한 오토마타(DFA) 기계라고 한다.

그렇기 때문에 컴퓨터는 ==난수 생성 알고리즘==을 통해 난수를 만든다.
이 또한 알고리즘이므로 정해진 입력에 정해진 출력을 만들어내는 것은 다를 바가 없다.

알고리즘의 입력으로 ==seed== 값을 지정해 줄 수 있는데, 
보통은 이 seed 값을 ms 단위의 시간으로 잡음으로써 우리가 볼 때는 마치 컴퓨터가 랜덤으로 난수를 선택하는 듯 보이게 한다.

## 실행 결과

```java
Random rand = new Random();  
rand.setSeed(LocalDate.of(2024, 1, 13).toEpochDay());  
  
for (int i = 0; i < 5; i++) {  
    System.out.println(rand.nextInt(10));  
}
```

```
2
9
6
7
3
```

위의 코드를 몇 번을 실행해봐도 항상 같은 값을 출력하게 된다.
왜냐하면 같은 seed 값을 입력으로 주었기 때문이다!

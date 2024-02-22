## 🔍Classpath?
---
 ![[Untitled.png]]

- Java 파일(`.java`) 컴파일 → 바이트 코드(`.class`) → JVM이 이를 읽어서 실행 → 기계어로 번역
- 자바로 작성된 프로그램을 컴파일(compile)하고 실행(run)할 때, **특정 경로에서부터 시작하여 클래스 파일과 패키지를 탐색**하게 됨 → **이를 classpath라고 한다.**

## 🔍compileClasspath vs runtimeClasspath
---

### compileClasspath

- Java 코드를 class 파일로 컴파일 할 때 탐색하는 경로
- **컴파일 시간**에 필요한 의존성, **빌드 결과물에 포함되지 않음**

### runtimeClasspath

![[Untitled 1.png]]

- class 파일을 JVM이 실행할 때 탐색하는 경로
    
- **런타임 시간**에 필요한 의존성, **빌드 결과물에 포함**됨
   
    
    - 일반적으로, **compileClasspath에는 runtimeClasspath에 포함되는 대부분의 의존성이 포함되어 있음**

<aside> 💡 컴파일 시점에만 필요한 의존성, 실행 시점에만 필요한 의존성이 있으므로 Gradle에서는 **의존성을 어느 범위로 노출시킬 것인지 결정**할 수 있게 한다.

</aside>

## 🔍compileOnly vs runtimeOnly
---

### compileOnly

- **라이브러리가 컴파일 시점에만 필요(컴파일 경로에 노출)**
- 프로젝트 빌드 시점에 해당 라이브러리를 컴파일에 사용, 빌드 결과물에는 포함X
- ex) lombok: 컴파일 시에만, getter, setter 등 필요한 것을 생성

### runtimeOnly

- **라이브러리가 런타임에만 필요(런타임 경로에 노출)**
- 프로젝트 빌드 시점에 해당 라이브러리를 classpath에 추가하지 않고, 런타임에 필요한 경우에만 포함
- 해당 클래스에서 코드 변경이 발생해도 컴파일 다시 할 필요없음
- ex) DB, log 관련 dependency

## 🔍전이 의존성(transitive dependency)
---

![[Untitled 2.png]]

- 모듈 C는 모듈 B에 의존, 모듈 B는 모듈 A에 의존
    - **모듈 C도 모듈 A에 의존하게 됨 → 이것을 전이 의존성이라고 한다.**
- api와 implementation 모두 컴파일 경로와 런타임 경로에 노출된다.
    - 그러나 둘 사이에는 **전이 의존성의 컴파일 경로 노출 여부**에 따른 차이가 있다.
    - 모듈 C에서 모듈 B를 사용할 때, 모듈 B에서 모듈 A를 사용하므로 전이 의존성의 런타임 경로에 당연히 노출된다.
![[Untitled 3.png]]
![[Untitled 4.png]]
![[Untitled 5.png]]

## 🔍api vs implementation
---

### **api(compile) → `java-library` 플러그인 필요!**

- 라이브러리가 컴파일 및 런타임 모두 필요(모든 경로에 노출)
- **전이 의존성을 허용**
    - 모듈 C에서는 implementation을 이용하여 모듈 B에 의존
    - 모듈 B에서는 api를 이용하여 모듈 A에 의존
    - **모듈 C에서 모듈 A의 코드에 접근할 수 있다!**
- A 모듈 수정 시, A를 의존하는 모든 모듈들이 rebuild

### ****************************implementation****************************

- 라이브러리가 컴파일 및 런타임 모두 필요(모든 경로에 노출)
- **********************************************************전의 의존성 사용 불가**********************************************************
    - 모듈 C에서는 implementation을 이용하여 모듈 B에 의존
    - 모듈 B에서는 implementation를 이용하여 모듈 A에 의존
    - **모듈 C에서 모듈 A의 코드에 접근이 불가능하다! → 컴파일 에러**
    - **컴파일 시간에 다른 모듈에 대한 종속성 드러내지 않음**
- A 모듈 수정 시, A를 직접적으로 의존하는 모듈만 rebuild

### implementation 사용의 장점

- 의존성이 해당 모듈을 사용하는 컴파일 클래스 경로에 노출되지 않음
    - 모듈 C에서는 모듈 A의 코드에 접근할 수 없다.
    - 예상치 못하게 전이 의존성에 빠질 일이 없다.
- classpath가 줄어들어 컴파일이 빨라진다.
- implementation하는 의존성들이 변경되어도, 재컴파일하지 않아도 된다.
- maven-publish → 알아보자

### 그렇다면 api는 언제 쓰는데?

- ABI (Application Binary Interface)
    - 부모 클래스 또는 인터페이스에 사용되는 타입
    - 제너릭 타입을 포함하여 public 메소드 파라미터로 사용하는 타입
    - 퍼블릭 필드에 사용되는 타입
    - 퍼블릭 애노테이션 타입

## 💡정리
---

![[Untitled 6.png]]

<aside> 💡 **최대한 implementation을 사용하고, api는 사용하지 말자**

</aside>

## 📜Ref
---

[[Gradle] Gradle Java 플러그인과 implementation와 api의 차이](https://mangkyu.tistory.com/296)

[[Spring] Gradle 파일 implementation, api, runtimeOnly, compileOnly... 등에 대해](https://bepoz-study-diary.tistory.com/372)

[The Java Library plugin configurations](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)
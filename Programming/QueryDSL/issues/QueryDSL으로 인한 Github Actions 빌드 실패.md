## Error

![[Pasted image 20240312004230.png]]

QueryDSL 관련 코드를 추가한 이후로 빌드 워크플로가 실패했다.

`Failed to create parent directory '/generated' when creating directory '/generated/querydsl'`

열심히 구글링해보니 `/generated` 폴더가 없거나, 생성할 수 있는 권한이 없어서 에러가 나는 것이라고 한다...
내 CI 워크플로 문제라고 인식하고 권한을 추가해보았다.

### 워크플로 작업에 대한 권한 변경

```yaml
jobs:  
  build:  
    runs-on: ubuntu-latest  
    permissions:  
      contents: write
```

이걸 바꾸면 폴더 생성 권한이 생기나?해서 한 번 해봤다.
결과는 실패...

저기에 있는 `permissons`는 Github에서 제공하는 REST API에 대한 권한이었다. (GET, POST 등등...)
ref) https://docs.github.com/ko/actions/using-jobs/assigning-permissions-to-jobs

### 디렉토리 + 하위 파일들 권한 변경

gradlew 프로세스가 디렉토리를 만들고 싶어도 접근 권한이 없기 때문에 실패했다고 생각해여
**프로젝트 폴더 + 하위의 모든 파일의 권한을 싹다 777로 변경했다!** (만약 된다고 해도 좋지 않은 선택같다...)

![[Pasted image 20240312005819.png|500]]

그러나 결과는 역시나 실패...


## 어쨌든...해결?

ref1) [Spring Data Mongo 프로젝트에 QueryDsl 적용하기](https://velog.io/@wwlee94/spring-mongodb-querydsl)
ref2) [그레이들 Annotation processor 와 Querydsl](https://honeymon.io/tech/2020/07/09/gradle-annotation-processor-with-querydsl.html)
구글링하다 눈알이 빠지기 전에 이런 보석같은 글을 발견했다!

요약하자면 우리는 gradle 5.0 이상의 버전을 사용 중이기 때문에, QClass의 생성 방법도 변경되었다.
Annotation Processor 기반의 설정 방식을 사용하기 때문에, `@Entity` 어노테이션이 명시되어 있는 경우, 자동으로 QClass가 생성된다.

### 기존의 `build.gradle` 스크립트 중

```
//Querydsl setting
def querydslDir = layout.buildDirectory.dir("/generated/querydsl")  
tasks.withType(JavaCompile) {  
	options.getGeneratedSourceOutputDirectory().set(file(querydslDir))
}  
sourceSets { 
	main.java.srcDirs += [ querydslDir ]
}  
clean { 
	delete file(querydslDir)
}
```

 `bulid/generated/querydsl` 폴더에 QClass들을 생성하기 위해 위와 같은 코드를 넣어주었는데, 필요가 없었던 것이다!

**폴더를 생성하는 코드를 제외**하면서 build도 성공적으로 끝났다.

![[Pasted image 20240312014651.png]]
야호~


## 해결은 했지만... 왜일까?

아직도 디렉토리 생성에 실패하는 이유를 알지 못했다...
보안 상의 이유로 쓰기 작업을 막아둔 것일까?
특별한 다른 방법을 써야 하는 것일까?

찜찜하다...
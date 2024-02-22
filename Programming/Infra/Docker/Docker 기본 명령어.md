
## 컨테이너 레벨 명령어

#### docker ps

#### Docker 컨테이너에 접속하기

``` bash
docker exec -it {컨테이너ID} /bin/bash
```

#### 사용하지 않는 컨테이너 일괄 삭제

``` bash
docker container prune
```

## 이미지

#### 설치된 이미지 리스트

``` bash
docker images
```

#### 이미지 삭제

``` bash
docker rmi {저장소명:태그}
```

## ETC

```
docker system prune -f
```

**주의!**
- 위 명령어를 사용하면 컨테이너 뿐만 아니라 network도 같이 삭제된다.
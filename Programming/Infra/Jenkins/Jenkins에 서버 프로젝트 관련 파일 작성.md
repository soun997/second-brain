
## [[Dockerfile]]

```Dockerfile
FROM amazoncorretto:17 AS builder
WORKDIR /app

COPY gradlew ./
COPY gradle ./gradle
COPY build.gradle ./
COPY settings.gradle ./
COPY src ./src

RUN chmod +x ./gradlew
RUN ./gradlew spotlessApply
RUN ./gradlew clean build -x test -x asciidoctor

FROM amazoncorretto:17
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","-Dspring.profiles.active=dev" ,"app.jar"]
```

## docker-compose.yml

```yml
version: "3"
services:
  spring:
    container_name: spring
    build:
      context: .
    environment:
      MARIA_URL: ${MARIA_URL}
      MARIA_USER: ${MARIA_USER}
      MARIA_PASSWORD: ${MARIA_PASSWORD}
      JWT_SECRET_KEY: ${SECRET_KEY}
    ports:
      - "8080:8080"
    networks:
      - deploy
    restart: always

networks:
  deploy:
    external: true
```

## .env

```env
MARIA_URL=jdbc:mariadb://veganlife.cxpkchuayi3r.ap-northeast-2.rds.amazonaws.com:3306/veganlife?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
MARIA_USER=admin
MARIA_PASSWORD=zhdrhrlwhdk

JWT_SECRET_KEY=7Jqw66as64qU67mE6rG07J2A7JWE64uI7KeA66eM67mE6rG065287J207ZSE66W87ISx6rO17KCB7Jy866Gc6rCc67Cc7ZWc64uk7ZmU7J207YyF
```
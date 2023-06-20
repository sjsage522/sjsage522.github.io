---
title: (Docker) 도커 튜토리얼, SpringBoot 애플리케이션을 도커를 통해 실행하기
author: jun
date: 2023-06-13 14:44:00 +0900
categories: [Docker]
tags: [docker, docker-compose]
render_with_liquid: false
comments: true
img_path: /assets/img/post/20230613/

---

## Image 빌드하기

### 프로젝트 클론

터미널에서 다음 명령을 통해 예제 프로젝트를 클론합니다.

```bash
cd /path/to/working/directory #원하는 위치에서
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
```



### 애플리케이션 실행 테스트 (Optional)

클론한 애플리케이션이 정상적으로 동작하는지 테스트 합니다. 해당 부분은 건너뛰셔도 상관없습니다.👇

> 예제 애플리케이션을 실행하기 위해서는 Java OpenJDK 버전 15 이상이 요구됩니다. ([Download and install Java](https://jdk.java.net/))



프로젝트를 클론한 위치에서 다음 명령을 입력합니다.

```bash
./mvnw spring-boot:run
```

<br/>

프로젝트에 필요한 디펜던시를 다운로드하고 프로젝트가 빌드 및 실행이 됐으면, 동작 여부를 브라우저에서 확인해 주세요. [http://localhost:8080](http://localhost:8080)

애플리케이션이 잘 작동한다면, `CTRL -c` 로 중지해주세요.

### Dockerfile 만들기

```dockerfile
# syntax=docker/dockerfile:1

FROM eclipse-temurin:17-jdk-jammy

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]
```

해당 Dockerfile은 base 이미지로 `eclipse-temurin:17-jdk-jammy`를 사용하여 Docker 컨테이너를 빌드하는 데 사용됩니다.

Dockerfile의 작업 디렉토리는 `/app`로 설정되어 있습니다. 모든 명령은 해당 디렉토리 내에서 실행됩니다.

이 Dockerfile에서는 다음과 같은 단계들이 수행됩니다:

1. `.mvn/` 디렉토리와 `mvnw`, `pom.xml` 파일을 작업 디렉토리(`/app`)로 복사합니다.
2. `./mvnw dependency:resolve` 명령을 실행하여  `pom.xml` 파일에 지정된 모든 디펜던시를 다운로드 합니다.
3. `src/` 디렉토리를 작업 디렉토리(`/app`)로 복사합니다.
4. CMD 명령을 통해 `./mvnw spring-boot:run` 명령을 실행하여 Spring Boot 애플리케이션을 실행합니다.

#### `.dockerignore` 파일 만들기

빌드 속도 향상 및 이미지 생성에 불필요한 파일을 포함시키지 않기위해서 `.dockerignore` 파일을 `Dockerfile` 파일의 위치에 생성해 줍니다.

파일 내용은 다음과 같습니다.

```
target
```

### Image 빌드

다음 명령으로 Doker Image를 빌드합니다.

`--tag`(`-t`) 옵션을 통해 이미지에 태그를 지정해줄 수 있습니다. 만약, 따로 태그명을 전달하지 않으면 `latest`로 지정됩니다.

```bash
docker build --tag java-docker .
```

<br/>

`docker images` 명령을 통해 이미지가 잘 빌드되었는지 확인할 수 있습니다.

``` bash
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED          SIZE
java-docker         latest              9607b4662f59        24 hours ago     629MB
```

## Image를 컨테이너로 실행하기

다음 명령을 통해 컨테이너를 실행합니다. `--publish`(`-p`) 옵션으로 내부,외부 포트를 연결해줍니다.

`--detach`(`-d`) 옵션을 함께 주면, 백그라운드에서 동작시킬 수 있습니다. 

```bash
docker run --publish 8080:8080 java-docker
```

<br/>

curl 명령을 통해 요청 및 응답이 정상인지 확인합니다.

```shell
curl --request GET \
--url http://localhost:8080/actuator/health \
--header 'content-type: application/json'
```

<br/>

`docker ps` 명령으로 실행중인 컨테이너에 대한 정보를 볼 수 있으며, `docker ps -a` 명령으로 모든 컨테이너들의 정보를 볼 수 있습니다.

```
docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                    NAMES
5ff83001608c   java-docker      "./mvnw spring-boot:…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   trusting_beaver
```

컨테이너를 시작할 때 따로 이름을 주지않으면 Docker는 임의의 이름을 생성합니다. (trusting_beaver)

`docker stop {NAMES}` 명령으로 컨테이너를 중지합니다.

<br/>

다음과 같은 명령으로 컨테이너를 생성하고 실행합니다. 이 때, `--name` 옵션을 통해 컨터이너의 이름을 지어줍니다.

>  `--rm` 옵션을 주면, 컨테이너가 종료될 때 컨테이너 관련된 리소스(파일 시스템, 볼륨 등)까지 전부 제거가 됩니다.

```bash
docker run --rm -d -p 8080:8080 --name springboot-server java-docker

docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
2e907c68d1c9   java-docker   "./mvnw spring-boot:…"   8 seconds ago   Up 8 seconds   0.0.0.0:8080->8080/tcp   springboot-server
```

`docker stop springboot-server` 명령으로 컨테이너 중지 후, `docker ps` 로 확인해보면, `--rm` 옵션으로 인해 컨테이너가 제거된 것을 볼 수 있습니다.

## 컨테이너에서 데이터베이스 실행하기

예제 애플리케이션은 DB로부터 데이터를 가져올 수 있어야합니다. 

먼저, 컨테이너에서 데이터베이스를 실행하고 볼륨과 네트워킹을 사용해 데이터를 유지 및 애플리케이션과 통신하는 방법을 알아보겠습니다. 

먼저 MySQL 관련 데이터를 **볼륨 마운트**하기 위한 볼륨을 생성합니다.

```bash
docker volume create mysql_data
docker volume create mysql_config
```

데이터베이스와 애플리케이션이 서로 통신하기 위한 네트워크를 만듭니다. 

```bash
docker network create mysqlnet
```

이제 컨테이너에서 MySQL을 실행하고 위에서 생성한 불륨과 네트워크를 연결합니다.

```bash
docker run -it --rm -d -v mysql_data:/var/lib/mysql \
-v mysql_config:/etc/mysql/conf.d \
--network mysqlnet \
--name mysqlserver \
-e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic \
-e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic \
-p 3306:3306 mysql:8.0
```

`Dockerfile`을 다음과 같이 수정하여, MySQL 스프링 profile을 활성화하여 H2 -> MySQL로 DB를 전환해줍니다.

```dockerfile
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql"]
```

이미지를 빌드해 줍니다.

```bash
docker build --tag java-docker .
```

이제 애플리케이션에 대한 컨테이너를 생성 및 실행합니다. MySQL에 접근할 수 있도록 환경변수도 설정해 줍니다.

```bash
docker run --rm -d \
--name springboot-server \
--network mysqlnet \
-e MYSQL_URL=jdbc:mysql://mysqlserver/petclinic \
-p 8080:8080 java-docker
```

실행 후, `docker logs -f springboot-server` 명령을 실행하여 로그를 확인할 수 있습니다.

실행이 완료되면, 다음 명령어로 테스트 해봅니다.

```bash
curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```

json 데이터가 정상적으로 반환되면 성공입니다.

## 개발환경을 위한 Dockerfile

개발환경과 운영환경을 위한 도커파일을 구성하겠습니다. 

해당 Dockerfile은 Maven을 사용하여 Spring Boot 애플리케이션을 빌드하고 실행하는 다단계 빌드(Multi-stage build)를 수행합니다. 

```dockerfile
# syntax=docker/dockerfile:1

FROM eclipse-temurin:17-jdk-jammy as base
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve
COPY src ./src

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM eclipse-temurin:17-jre-jammy as production
EXPOSE 8080
COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

다음은 Dockerfile의 각 부분에 대한 설명입니다.

1. `FROM eclipse-temurin:17-jdk-jammy as base`: `eclipse-temurin:17-jdk-jammy` 이미지를 base 이미지로 선택합니다.  `base`라는 이름의 이미지로 사용합니다.
2. `FROM base as development`: `base`이미지를 기반으로 `development`라는 이름의 중간 단계 이미지를 생성합니다.
3. `CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]`: 개발 환경에서 실행되는 컨테이너에서 실행될 명령을 정의합니다. `./mvnw spring-boot:run` 명령을 실행하며, MySQL 프로필과 디버깅을 위한 JVM 인자를 설정합니다.
4. `FROM base as build`: `base` 이미지를 기반으로 `build`라는 이름의 중간 단계 이미지를 생성합니다.
5. `RUN ./mvnw package`: Maven을 사용하여 애플리케이션을 패키지화(빌드)합니다. 이 명령은 JAR 파일을 생성합니다.
6. `FROM eclipse-temurin:17-jre-jammy as production`: `eclipse-temurin:17-jre-jammy` 이미지를 base 이미지로 선택합니다. `production`이라는 이름의 이미지로 사용됩니다.
7. `EXPOSE 8080`: 컨테이너가 노출할 포트 번호를 지정합니다.
8. `COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar`: `build` 단계에서 생성된 JAR 파일을 이미지로 복사합니다.
9. `CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]`: 이미지에서 실행될 명령을 정의합니다. Java 명령을 사용하여 JAR 파일을 실행합니다.

### Docker Compose 구성하기

프로젝트 루트 디렉토리에서 `docker-compose.dev.yml` 파일을 만들고 다음과 같이 작성합니다.

```yaml
version: '3.8'
services:
  petclinic:
    build:
      context: .
      target: development
    ports:
      - "8000:8000"
      - "8080:8080"
    environment:
      - SERVER_PORT=8080
      - MYSQL_URL=jdbc:mysql://mysqlserver/petclinic
    volumes:
      - ./:/app
    depends_on:
      - mysqlserver

  mysqlserver:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_USER=petclinic
      - MYSQL_PASSWORD=petclinic
      - MYSQL_DATABASE=petclinic
    volumes:
      - mysql_data:/var/lib/mysql
      - mysql_config:/etc/mysql/conf.d
volumes:
  mysql_data:
  mysql_config:
```

다음 명령을 통해 docker-compose 파일을 실행합니다.

```bash
docker-compose -f docker-compose.dev.yml up --build
```

> --build 옵션을 통해 이미지에 변경사항이 있을 경우, 컨테이너를 시작하기 전에 이미지를 빌드합니다.
> <br/>만약, 해당 옵션이 없으면 이전에 빌드된 이미지를 사용하게 됩니다.

애플리케이션이 실행됐으면, 다음 명령을 입력해 응답이 정상적으로 오는지 확인해봅니다.

```bash
curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```

### IDE(인텔리제이)에서 디버깅하기

IntelliJ IDEA 에서 제공하는 디버거를 사용하여 디버깅을 해보겠습니다. IntelliJ IDEA에서 예제 프로젝트를 열고, **Run** 메뉴 > **Edit Configuration** 으로 이동합니다.

다음과 같이 디버그용 설정을 추가해 줍니다.

![image-20230615094356634](/image-20230615094356634.png)

종단점을 설정하고, 디버깅을 시작해 줍니다.

![image-20230615094914263](/image-20230615094914263.png)

다음과 같은 명령을 실행합니다.

```bash
curl --request GET --url http://localhost:8080/vets
```

아래 그림과 같이 디버깅이 잘 수행된 것을 확인할 수 있습니다.

![image-20230615095008995](/image-20230615095008995.png)

또한, [SpringBoot Dev Tools](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.remote-applications) 를 이용하여 개발할 수도 있습니다. 

> SpringBoot Dev Tools는, 애플리케이션을 개발하기 위한 유용한 도구를 제공합니다. 
>
> 주요 기능으로는, classpath에 존재하는 파일의 변경을 감지하고 자동으로 서버를 재시작, 템플릿 엔진의 캐시기능 비활성화, Hot reload 기능 등이 있습니다.

## 테스트 실행하기

애플리케이션에서 정의된 테스트 코드를 Docker에서 수행하는 방법을 알아보겠습니다. 

### Dockerfile 리팩토링

Spring Pet Clinic 프로젝트에는 이미 테스트 코드들을 정의되어 있습니다. 다음 명령을 통해 컨테이너를 생성 및 시작하고 테스트를 수행할 수 있습니다.

```bash
docker run -it --rm --name springboot-test java-docker ./mvnw test
```

Dockerfile을 실행하여, 테스트 이미지를 빌드하기 위해 Dockerfile을 다음과 같이 수정합니다.

```dockerfile
# syntax=docker/dockerfile:1

FROM eclipse-temurin:17-jdk-jammy as base
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve
COPY src ./src

FROM base as test
CMD ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package


FROM eclipse-temurin:17-jre-jammy as production
EXPOSE 8080
COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

`test` 단계가 추가된 것을 확인할 수 있습니다.

이미지를 다시 빌드하고 테스트를 실행하겠습니다. 이번에는 다단계 빌드에서 test 단계만을 빌드하기 위해 `--target test`라는 옵션을 주도록 하겠습니다.

```bash
docker build -t java-docker --target test .
```

빌드가 완료되면, 컨테이너를 생성 및 실행하여 테스트가 통과되는지 확인합니다.

```bash
docker run -it --rm --name springboot-test java-docker
```

지금의 방식에서는 테스트를 진행하기 위해서 이미지 빌드와 컨테이너 실행이라는 두 번의 명령이 필요합니다. Dockerfile 에서 `CMD` 대신 `RUN` 키워드를 사용하면, 이미지를 빌드할 때 해당 명령이 수행됩니다.

> CMD 키워드는 이미지 빌드 중에는 명령이 수행되지 않고, 컨테이너에서 이미지를 실행할 때 수행됩니다.

다음과 같이 Dockerfile을 수정합니다.

```dockerfile
# syntax=docker/dockerfile:1

FROM eclipse-temurin:17-jdk-jammy as base
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:resolve
COPY src ./src

FROM base as test
RUN ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM eclipse-temurin:17-jre-jammy as production
EXPOSE 8080
COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

이제 테스트를 실행하려면 다음과 같이 명령을 실행하기만 하면 됩니다.

```bash
docker build -t java-docker --target test .
```

## 마침

> 궁금하시거나, 부족한 내용이 있다면 코멘트 부탁드립니다. 🙇🏻‍♂️

## 레퍼런스

Docker docs<br/>

[https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)

[https://docs.docker.com/language/java/](https://docs.docker.com/language/java/)

Spring Boot DevTools<br/>

[https://velog.io/@bread_dd/Spring-Boot-Devtools](https://velog.io/@bread_dd/Spring-Boot-Devtools)

[https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.remote-applications](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools.remote-applications)


# DAY 60

## 목차
- [컨테이너화](#컨테이너화)

---

compile 실행 가능한 파일 만들기<br>
build 의존성을 포함한 compile된 파일들을 패키징 하는 것<br>
`java -jar app.jar`로 실행

## 컨테이너화
#### 전통적인 배포
- CI/CD 도구 없이 수동으로 서버에 직접 애플리케이션을 배포하고 운영하는 방식
- 로컬에서 빌드된 .jar 또는 .war 파일을 수동으로 서버에 업로드
- 서버내 JDK, WAS, DB 등을 수동 설치 및 설정
- 직접 쉘 명령어로 실행 및 관리
##### intellij
- Tasks -> build -> :bootjar = 바로 실행 가능한 jar 파일 생성
- Tasks -> build -> :build = 라이브러리 다 뺀 plain jar 파일 생성

- PaaS : Platform
- IaaS : Infrastructure
- SaaS : Software

- 문제점
  - 환경 의존성 문제
    - 운영 황경과 개발 환경 사이의 격차
  - 설치 과정의 복잡성 및 반복성
    - 모든 의존 구성 요소를 직접 설치 해야 함
  - 서버 자원 활용의 비효율성
    - 단일 서버, 단일 인스턴스 구조로 되어 있어 확장성이 떨어짐
    - 트래픽이 증가하면 추가 서버를 직접 구성해야 하며, 로드밸런싱, 무중단 배포가 어렵다.
  - 배포 오류 및 반복 작업 증가
    - 매번 수동 반복 -> 매우 비효율적
  - 자동화 부재로 인한 유지보수 비용 증가
    - 가시성 부족, 로그 (기록) 남기기 어려움

#### 컨테이너
- 운영체제 수준에서 격리된 실행 환경
- 컨테이너 내부에서 실행되는 애플리케이션은 마치 독립된 OS 위에서 실행되는 것처럼 작동하지만, 실제로는 호스트 OS의 커널을 공유합니다.
- 특징
  - 하나의 컨테이너는 하나의 프로세스를 실행하는 데 최적화됨
  - 다른 컨테이너와 파일 시스템, 네트워크, 환경 변수를 격리해서 사용함
  - 가볍고 빠르게 실행 됨
 
#### 도커 컨테이너
- 컨테이너 기술을 손쉽게 사용할 수 있도록 만들어진 오픈소스 플랫폼
- 특성
  - Docker 이미지로부터 생성됨
  - 독립적인 환경을 제공하며, 다른 컨테이너나 호스트와 격리됨
  - 실행이 매우 빠르고 자원 사용이 적음
  - 시스템 전체가 아니라 애플리케이션 단위로 패키징되어 있음

#### 도커 이미지
- 컨테이너를 실행하기 위한 정적인 실행 패키지
- 컨테이너의 설계도
- 포함 요소
  - 운영체제의 최소 구성 요소 (Ubuntu minimal, Alpine 등)
  - 애플리케이션 실행 파일
  - 의존 라이브러리, 설정 파일, 환경 변수
  - Dockerfile에 정의된 명령들
  - ```
    # Dockerfile 예시
    FROM openjdk:17
    COPY ./build/libs/app.jar /app/app.jar
    ENTRYPOINT ["java", "-jar", "/app/app.jar"]
    ```
- 특징
  - 불변 : 한 번 생성되면 수정 불가
  - 레이어 구조 : 효율적인 저장 및 캐시 활용 가능
  - 재사용 가능 : 동일 이미지로 수백 개의 컨테이너 생성 가능
 
- 운영 환경, 개발 환경, 테스트 환경이 모두 다르더라도 "컨테이너 안에서는 항상 동일하게 동작

#### 문제 해결
- 실행 환경 표준화
- 배포 자동화
  - CI/CD 파이프라인과 뛰어난 연동성
  - 흐름 예시(CI 서버 자동 동작 수행)
    - 단위 테스트 및 통합 테스트 실행
    - Dockerfile 기반으로 이미지 빌드
    - Docker Hub 또는 AWS ECR 등으로 이미지 푸시
    - 서버 또는 쿠버네티스 클러스터에서 최신 이미지로 재배포
- 자원 활용의 효율화
  - 호스트 OS의 커널 공유
  - 자원 제한 옵션 가능

## Docker
- 주요 구성 요소
  - 이미지 : 실행 환경 전체를 코드처럼 패키징한 템플릿
  - 컨테이너 : 이미지 기반으로 실행되는 격리된 프로세스 환경
  - Dockerfile : 이미지를 생성하기 위한 선언형 스크립트
  - Docker CLI /Daemon
  - ```
    # 예시: Dockerfile 기반으로 컨테이너 실행하기
    $ docker build -t my-app .         # Dockerfile로부터 이미지 생성
    $ docker run -d -p 8080:8080 my-app  # 생성된 이미지로 컨테이너 실행
    ```
- 특징과 장점
  - 실행 환경의 이식성
  - 빠른 컨테이너 기동 속도
  - 경량성 및 자원 효율성
  - 자동화 도구와의 탁월한 연계성
  - **마이크로서비스와의 찰떡궁합**
  - 오픈 에코시스템과 커뮤니티
 
#### DockerHub
- Docker에서 운영하는 공식 이미지 저장소
- 제공 기능
  - 이미지를 Pull/Push 할 수 있는 저장소 제공
  - 공식 이미지와 사용자 이미지 구분
  - 자동 빌드 가능
  - 이미지 버전 관리 및 태그 관리 기능
  - 팀 및 조직 단위 협업 가능
  - ```
    # DockerHub에서 nginx 이미지 받아 실행하기
    $ docker pull nginx                # 이미지 다운로드
    $ docker run -d -p 80:80 nginx     # 컨테이너 실행
    ```
- 도커 저장소
  - 종류
    - Docker Hub
    - Private Registry
    - Public Registry
  - 구성 요소
    - Repository : 이미지 저장 단위
    - Tag : 이미지 버전 구분
    - Official Image : Docker가 직접 관리하는 신뢰성 높은 이미지
    - User Image : 사용자가 업로드한 커스텀 이미지
    - Automated Build : GitHub와 연동해 자동으로 이미지를 빌드하는 기능
    - Organization : 팀 단위 이미지 관리 기능
   
#### Docker 명령어
- docker pull 이미지 다운로드 명령어
  - ```
    docker pull nginx
    # 가장 최신 버전
    ```
  - ```
    docker pull nginx:1.25
    # 1.25 태그가 붙은 버전
    ```
- docker images 이미지 목록 조회
  - docker images --filter "dangling=true" 태그가 없는 이미지 목록(none:none)만 출력
    - 보통 docker build 도중 실패하거나 중간 단계에서 생성된 이미지들
  - docker images --format '{{json .}}' json 형태로 출력 가능 (스크립트 용도)
- docker rmi 이미지 삭제
  - id로 삭제, repository:TAG 형식으로 삭제 가능.
  - docker rm 은 컨테이너 삭제
  - 강제 삭제 옵션 f
  - ```
    docker rmi -f nginx
    컨테이너에서 사용 중이더라도 삭제됨
    ```
  - docker imaga prune
    - dangling 이미지 삭제
    - 삭제 확정 시 y 또는 -force 필요
- docker run 컨테이너 실행 / 이미지로부터 새로운 컨테이너를 만들고 실행
  - `-d` 백그라운드 실행
  - `--name` 컨테이너 이름 지정
  - `-p` 포트 매핑
- docker ps 컨테이너 상태 확인 / 현재 실행중인 컨테이너 목록 표시
  - docker ps 실행중인 컨테이너
  - docker ps -a 모든 컨테이너 보기 (중지된 것 포함)
- docker start/stop/restart 컨테이너 시작, 중지, 재시작
  - start 중지된 컨테이너 실행
  - stop 실행 중인 컨테이너 중지
  - restart 실행 중인 컨테이너 재시작
- docker exec 실행 중인 컨테이너 안에서 명령 실행 가능하게
  - docker exec -it [container name] bash
- docker logs 컨테이너 로그 확인
  - docker logs -f [container name] 실시간 로그보기
- docker rm 컨테이너 삭제 / 중지된 컨테이너만 삭제 가능
  - docker rm -f 실행 중 컨테이너 강제 삭제

#### docker 네트워크
- docker는 연결을 위해 **네트워크 드라이버** 라는 방식을 사용
- 기본 네트워크 종류
  - bridge(기본) : docker가 가상 브릿지(스위치)를 만들고, 모든 컨테이너를 여기에 연결. 외부 접근은 포트 매핑 필요.
    - 컨테이너 간 통신 가능
    - 외부 노출 제어 가능
    - 포트 매핑 안 하면 외부에서 접근 불가
  - host : 컨테이너가 호스트 pc의 네트워크를 그대로 사용. IP도 호스트와 동일
    - 네트워크 성능 우수
    - 포트 매핑 필요 없음
    - 포트 충돌 가능성 큼
    - 컨테이너 격리 약화
  - none : 네트워크 없음. 외부, 내부 모두 통신 가능
    - 완전 격리
- 같은 브릿지 네트워크에 있는 컨테이너는 컨테이너 이름으로 서로 통신 가능
- docker network create my-net 네트워크 생성
- ```
  docker run -d --name container-a --network my-net nginx
  docker run -d --name container-b --network my-net alpine sleep 1000
  # 동일한 네트워크에 컨테이너 실행
  ```
- docker exec -it cotainer-b ping container-a 네트워크를 통한 통신 확인

- 포트 바인딩
  - 호스트의 포트를 컨테이너의 포트에 연결하여 외부에서 접근 가능하게 하는 방법
  - ```
    # 호스트 8080 → 컨테이너 80 포트 연결
    docker run -d --name web -p 8080:80 nginx
    ```
  - docker ps 바인딩 확인

#### 볼륨
- 컨테이너의 데이터를 영구적으로 저장하고
- 컨테이너 간에 데이터를 공유하기 위한 저장소
- 컨테이너가 삭제되더라도 볼륨에 저장된 데이터는 유지

#### dockerfile
- docker 이미지를 만들기 위한 명렁ㅇ 모음
- 각 명령어는 레이어를 만들고, 빌드 시 캐시가 재사용 됨
- 위쪽 명령이 바뀌면 아래쪽 캐시가 전부 무효 -> 빈번히 바뀌는 파일(copy 소스 등)은 아래쪽에 두면 빌드가 빠름

- 예시
- ```
  # 1) 베이스 이미지 선택 (OpenJDK 17)
  FROM eclipse-temurin:17-jdk
  
  # 2) 작업 디렉토리 설정
  WORKDIR /app
  
  # 3) 애플리케이션 파일 복사
  COPY build/libs/my-app-0.0.1-SNAPSHOT.jar app.jar
  
  # 4) 포트 노출 (Spring Boot 기본 포트)
  EXPOSE 8080
  
  # 5) 컨테이너 실행 시 명령어 지정
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```

- 명령어
  - FROM 기반이 되는 베이스 이미지를 지정
  - RUN 이미지 빌드 시 실행할 명령어
  - COPY 호스트의 파일/디렉토리를 이미지로 복사
  - WORKDIR 작업 디렉토리 지정
  - CMD 컨테이너 실행 시 기본 명령 지정
  - ENTRYPOINT 실행 진입점 설정 (CMD와 병행 가능)
  - EXPOSE 컨테이너가 사용하는 포트 명시
  - ENV 환경변수 설정
- 레이어
  - Docker 이미지는 여러 읽기 전용 레이어로 구성
  - dockerfile의 각 명령어가 하나의 레이어를 형성
  - 레이어는 캐시가 되어, 변경되지 않은 부분은 재사용
  - ```
    # 1) 베이스 이미지 선택 (OpenJDK 17)
    FROM eclipse-temurin:17-jdk
    
    # 2) 작업 디렉토리 설정
    WORKDIR /app
    
    # 3) 애플리케이션 파일 복사
    COPY build/libs/my-app-0.0.1-SNAPSHOT.jar app.jar
    
    # 4) 포트 노출 (Spring Boot 기본 포트)
    EXPOSE 8080
    
    # 5) 컨테이너 실행 시 명령어 지정
    ENTRYPOINT ["java", "-jar", "app.jar"]
    
    ------------------
    Layer 5   ENTRYPOINT
    ------------------
    Layer 4   EXPOSE
    ------------------
    Layer 3   COPY
    ------------------
    Layer 2   WORKDIR
    ------------------
    Layer 1   FROM
    ------------------
    ```
- 변경된 레이어와 그 이후의 모든 레이어를 새로 빌드 -> 속도 느려짐
- 수정이 빈번하면 아래쪽에 두는 것으로 빌드 속도 ⬆

- 4-2. 최적화 이후 Dockerfile (멀티스테이지 + 의존성 캐시 고정)
- ```
  # 빌드 단계: 도구/캐시 전용
  FROM gradle:8.8-jdk17 AS builder
  WORKDIR /build
  
  # 1) 변경 빈도 낮은 Gradle 메타만 먼저 복사 → 의존성 캐시 적중률↑
  COPY gradlew gradlew
  COPY gradle gradle
  COPY settings.gradle settings.gradle
  COPY build.gradle build.gradle
  RUN ./gradlew --no-daemon build -x test || true \
   && ./gradlew --no-daemon dependencies || true
  
  # 2) 소스는 나중에 복사 → 코드 변경에도 의존성 재다운로드 방지
  COPY src ./src
  RUN ./gradlew --no-daemon clean bootJar -x test
  
  # 실행 단계: 경량 런타임만 포함
  FROM eclipse-temurin:17-jre
  WORKDIR /app
  COPY --from=builder /build/build/libs/*.jar app.jar
  EXPOSE 8080
  ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
  ENTRYPOINT ["sh", "-lc", "exec java $JAVA_OPTS -jar app.jar"]
  ```
- 의존성 캐시 고정 -> 빌드시간 단축
- 런타임(JRE)+ 단일 JAR만 포함 -> 이미지 용량 다운
- 단계 분리로 안정성 확보 -> 환경 일관성 강화


# HyperFrame-Pinpoint

- 업로드된 바이너리는 HyperFrame Pinpoint 제품 설치를 위한 파일

<br>

## 설치 파일

### Hbase

- Verison : hbase-1.2.7-bin.tar.gz
- Note : [https://archive.apache.org/dist/hbase/1.2.7/](https://archive.apache.org/dist/hbase/1.2.7/)
- Pinpoint로부터 수집된 데이터 저장

### Pinpoint Collector

- Version : pinpoint-collector-boot-2.3.3.jar
- Note : [https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3](https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3)
- Agent의 데이터 수집

### Pinpoint Web

- version : pinpoint-web-boot-2.3.3.jar
- Note : [https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3](https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3)
- 모니터링 용 Web UI 제공

### Pinpoint Agent

- version : pinpoint-agent-2.3.3.tar.gz
- Note : [https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3](https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.3.3)
- 모니터링 할 Application의 데이터 계측

### Pinpoint 지원 Java Version

- java7 이상 지원

<br>

## 검증 환경

- CentOS Linux release 7.9.2009
- CentOS Stream release 8

<br>

## 설치 및 실행

### 1) Hbase 압축 풀기

```
$ cd ${INSTALLER_HOME}
$ tar -zxf hbase-1.2.7-bin.tar.gz

```

### 2) Hbase 환경 설정

```
$ cd ${HBASE_HOME}/conf
$ vi hbase-env.sh
...
# The java implementation to use. Java 1.7+ required.
export JAVA_HOME=${JAVA_HOME}
...

```

### 3) Hbase 실행

```
$ cd ${HBASE_HOME}/bin
$ ./start-hbase.sh

```

### 4) Hbase 스키마 생성

### 4-1. 아래 경로에 hbase-create.hbase 첨부파일 다운로드 (hbase-create.hbase 스크립트 없을 시)

```
  $ ${HBASE_HOME}/bin/hbase-create.hbase

```

### 4-2. 스키마 생성

```
$ cd ${HBASE_HOME}/bin
$ ./hbase shell ./hbase-create.hbase

```

### 5) Pinpoint Collector 실행

```
$ cd ${COLLECTOR_HOME}
$ java -jar -Dpinpoint.zookeeper.address=${PINPOINT_SERVER_IP} pinpoint-collector-boot-2.3.3.jar

```

### 6) Pinpoint Web 실행

```
$ cd ${WEB_HOME}
$ java -jar -Dpinpoint.zookeeper.address=${PINPOINT_SERVER_IP} pinpoint-web-boot-2.3.3.jar

port 변경하여 실행하는 경우 (기본 port는 8080)
$ java -jar -Dpinpoint.zookeeper.address=${PINPOINT_SERVER_IP} -Dserver.port=${PORT} pinpoint-web-boot-2.3.3.jar

```

### 7) Pinpoint Agent 압축 풀기

```
$ cd ${INSTALLER_HOME}
$ tar -zxf pinpoint-agent-2.3.3.tar.gz

```

### 8) Pinpoint Agent 환경 설정

```
$ cd ${AGENT_HOME}
$ vi pinpoint-root.config
...
profiler.transport.grpc.collector.ip=${PINPOINT_SERVER_IP}
...
profiler.collector.ip=${PINPOINT_SERVER_IP}
...

```

```
$ cd ${AGENT_HOME}/local
$ vi pinpoint.config
...
profiler.transport.grpc.collector.ip=${PINPOINT_SERVER_IP}
...
profiler.collector.ip=${PINPOINT_SERVER_IP}
```

```
$ cd ${AGENT_HOME}/release
$ vi pinpoint.config
,,,
profiler.transport.grpc.collector.ip=${PINPOINT_SERVER_IP}
...
profiler.collector.ip=${PINPOINT_SERVER_IP}
```

### 9) 네트워크 테스트

```
$ cd ${AGENT_HOME}/script
$ ./networktest.sh
```

### 10) 애플리케이션 Agent 연결 및 실행

```
$ java -jar -javaagent:${AGENT_HOME}/pinpoint-bootstrap-2.3.3.jar \\
  -Dpinpoint.agentId=${AGENT_ID} \\ # 고유 ID
  -Dpinpoint.applicationName=${APPLICATION_NAME} \\ # 그룹 NAME
  ${APPLICATION_HOME}/${APPLICATION}

```

### 11) 애플리케이션 삭제

```
http://${PINPOINT_SERVER_IP}:${PORT}/admin/removeApplicationName.pinpoint?applicationName=${APPLICATION_NAME}&password=admin
```

<br>

## Tomcat Agent 연동

### 1) Tomcat 설치

- 사전에 Tomcat 설치 필요
    
    [https://github.com/TmaxSoftOfficial/HyperFrame-Tomcat](https://github.com/TmaxSoftOfficial/HyperFrame-Tomcat) ([README.MD](http://readme.md/) 파일 참고)
    

### 2) [catalina.sh](http://catalina.sh/) 옵션 추가 or [setenv.sh](http://setenv.sh) 옵션 설정

1. [catalina.sh](http://catalina.sh) 옵션 추가

```
$ cd ${TOMCAT_HOME}/bin
$ vi catalina.sh
...
CATALINA_OPTS="$CATALINA_OPTS -javaagent:${AGENT_HOME}/pinpoint-bootstrap-2.3.3.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=${AGENT_ID}" # 고유 ID
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=${APPLICATION_NAME}" # 그룹 NAME
...

```

2. [setenv.sh](http://setenv.sh) 옵션 설정

```
$ cd ${TOMCAT_HOME}/bin
$ vi setenv.sh
...
AGENT_PATH="${AGENT_HOME}" # ex) "/usr/local/pinpoint-agent-2.3.3"
CATALINA_OPTS="$CATALINA_OPTS -javaagent:$AGENT_PATH/pinpoint-bootstrap.jar"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=${AGENT_ID}" # 고유 ID
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=${APPLICATION_NAME}" # 그룹 NAME
```

### 3) Tomcat 실행

```
$ cd {TOMCAT_HOME}/bin
$ ./startup.sh

```

### 4) 연동 및 모니터링 확인

- 기동 순서는 Pinpoint Collector, Pinpoint Web, Tomcat(Pinpoint Agent) 순서이다

<br>

## Pinpoint Web 접속

- `http://${PINPOINT_SERVER_IP}:${PORT}`
- ${PORT}는 Pinpoint Web 실행 시 지정한 port / default: 8080

<br>

## 웹 정보

### Server Map

- 애플리케이션 구성을 한 눈에 파악
- 노드 간에 호출된 건 수를 파악
- 필터를 통한 특정 요청에 대해서 조회 가능

### CallStack

- API 호출 정보 시각화
    - Response Summary (응답 결과 요약)
    - Response Avg & Max
    - Load (시간별 트랜잭션 응답 결과)
    - Load Avg & Max
- CallStack Trace
    - Trace View를 제공하여 오류나 병목 발생 지점 발견 가능

### Inspector

- 실시간 운영 환경 정보
    - CPU 사용량
    - 메모리 사용량
    - 스레드 개수
    - etc
        
<br>

## 로그 정보

### Pinpoint 로그 경로

- Pinpoint Collector
    
    ```
    ${COLLECTOR_HOME}/logs/pinpoint-collector.log
    
    ```
    
- Pinpoint Web
    
    ```
    ${WEB_HOME}/logs/pinpoint-web.log
    
    ```
    
- Pinpoint Agent
    
    ```
    ${AGENT_HOME}/logs/${AGENT_ID}/pinpoint_satat.log
    
    ${AGENT_HOME}/logs/${AGENT_ID}/pinpoint.log
    
    ```

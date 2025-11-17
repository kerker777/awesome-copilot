---
mode: 'agent'
description: 'Create Spring Boot Java Project Skeleton'
---

# 建立 Spring Boot Java 專案提示詞

- 請確保您的系統已安裝以下軟體：

  - Java 21
  - Docker
  - Docker Compose

- 如果您需要自訂專案名稱，請在[下載 Spring Boot 專案樣板](./create-spring-boot-java-project.prompt.md#download-spring-boot-project-template)中修改 `artifactId` 和 `packageName`

- 如果您需要更新 Spring Boot 版本，請在[下載 Spring Boot 專案樣板](./create-spring-boot-java-project.prompt.md#download-spring-boot-project-template)中修改 `bootVersion`

## 檢查 Java 版本

- 在終端機執行下列指令，檢查 Java 版本

```shell
java -version
```

## 下載 Spring Boot 專案樣板

- 在終端機執行下列指令，以下載 Spring Boot 專案樣板

```shell
curl https://start.spring.io/starter.zip \
  -d artifactId=${input:projectName:demo-java} \
  -d bootVersion=3.4.5 \
  -d dependencies=lombok,configuration-processor,web,data-jpa,postgresql,data-redis,data-mongodb,validation,cache,testcontainers \
  -d javaVersion=21 \
  -d packageName=com.example \
  -d packaging=jar \
  -d type=maven-project \
  -o starter.zip
```

## 解壓縮下載的檔案

- 在終端機執行下列指令，以解壓縮下載的檔案

```shell
unzip starter.zip -d ./${input:projectName:demo-java}
```

## 移除下載的壓縮檔

- 在終端機執行下列指令，以刪除下載的壓縮檔

```shell
rm -f starter.zip
```

## 變更目錄到專案根目錄

- 在終端機執行下列指令，以變更目錄到專案根目錄

```shell
cd ${input:projectName:demo-java}
```

## 新增額外的相依性

- 在 `pom.xml` 檔案中插入 `springdoc-openapi-starter-webmvc-ui` 和 `archunit-junit5` 相依性

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.8.6</version>
</dependency>
<dependency>
  <groupId>com.tngtech.archunit</groupId>
  <artifactId>archunit-junit5</artifactId>
  <version>1.2.1</version>
  <scope>test</scope>
</dependency>
```

## 新增 SpringDoc、Redis、JPA 和 MongoDB 設定

- 在 `application.properties` 檔案中插入 SpringDoc 設定

```properties
# SpringDoc configurations
springdoc.swagger-ui.doc-expansion=none
springdoc.swagger-ui.operations-sorter=alpha
springdoc.swagger-ui.tags-sorter=alpha
```

- 在 `application.properties` 檔案中插入 Redis 設定

```properties
# Redis configurations
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=rootroot
```

- 在 `application.properties` 檔案中插入 JPA 設定

```properties
# JPA configurations
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=rootroot
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

- 在 `application.properties` 檔案中插入 MongoDB 設定

```properties
# MongoDB configurations
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.username=root
spring.data.mongodb.password=rootroot
spring.data.mongodb.database=test
```

## 新增 `docker-compose.yaml` 檔案（包含 Redis、PostgreSQL 和 MongoDB 服務）

- 在專案根目錄建立 `docker-compose.yaml` 檔案，並新增下列服務：`redis:6`、`postgresql:17` 和 `mongo:8`。

  - Redis 服務應該具備
    - 密碼 `rootroot`
    - 將連接埠 6379 對應到 6379
    - 將磁碟區 `./redis_data` 掛載到 `/data`
  - PostgreSQL 服務應該具備
    - 密碼 `rootroot`
    - 將連接埠 5432 對應到 5432
    - 將磁碟區 `./postgres_data` 掛載到 `/var/lib/postgresql/data`
  - MongoDB 服務應該具備
    - 初始化根使用者名稱 `root`
    - 初始化根使用者密碼 `rootroot`
    - 將連接埠 27017 對應到 27017
    - 將磁碟區 `./mongo_data` 掛載到 `/data/db`

## 新增 `.gitignore` 檔案

- 在 `.gitignore` 檔案中插入 `redis_data`、`postgres_data` 和 `mongo_data` 目錄

## 執行 Maven 測試指令

- 執行 Maven 清理測試指令，以檢查專案是否正常運作

```shell
./mvnw clean test
```

## 執行 Maven 執行指令（可選）

- （可選）執行 `docker-compose up -d` 啟動服務、`./mvnw spring-boot:run` 執行 Spring Boot 專案、`docker-compose rm -sf` 停止服務。

## 讓我們逐步進行

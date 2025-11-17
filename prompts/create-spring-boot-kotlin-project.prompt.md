---
mode: 'agent'
description: 'Create Spring Boot Kotlin Project Skeleton'
---

# 建立 Spring Boot Kotlin 專案提示

- 請確保您的系統已安裝以下軟體：

  - Java 21
  - Docker
  - Docker Compose

- 如果需要自訂專案名稱，請在 [下載 Spring Boot 專案樣板](#下載-spring-boot-專案樣板) 中修改 `artifactId` 和 `packageName`

- 如果需要更新 Spring Boot 版本，請在 [下載 Spring Boot 專案樣板](#下載-spring-boot-專案樣板) 中修改 `bootVersion`

## 檢查 Java 版本

- 在終端機中執行下列命令並檢查 Java 版本

```shell
java -version
```

## 下載 Spring Boot 專案樣板

- 在終端機中執行下列命令以下載 Spring Boot 專案樣板

```shell
curl https://start.spring.io/starter.zip \
  -d artifactId=${input:projectName:demo-kotlin} \
  -d bootVersion=3.4.5 \
  -d dependencies=configuration-processor,webflux,data-r2dbc,postgresql,data-redis-reactive,data-mongodb-reactive,validation,cache,testcontainers \
  -d javaVersion=21 \
  -d language=kotlin \
  -d packageName=com.example \
  -d packaging=jar \
  -d type=gradle-project-kotlin \
  -o starter.zip
```

## 解壓縮已下載的檔案

- 在終端機中執行下列命令以解壓縮已下載的檔案

```shell
unzip starter.zip -d ./${input:projectName:demo-kotlin}
```

## 移除已下載的 zip 檔案

- 在終端機中執行下列命令以刪除已下載的 zip 檔案

```shell
rm -f starter.zip
```

## 解壓縮已下載的檔案

- 在終端機中執行下列命令以解壓縮已下載的檔案

```shell
unzip starter.zip -d ./${input:projectName:demo-kotlin}
```

## 新增額外的相依項目

- 在 `build.gradle.kts` 檔案中插入 `springdoc-openapi-starter-webflux-ui` 和 `archunit-junit5` 相依項目

```gradle.kts
dependencies {
  implementation("org.springdoc:springdoc-openapi-starter-webflux-ui:2.8.6")
  testImplementation("com.tngtech.archunit:archunit-junit5:1.2.1")
}
```

- 在 `application.properties` 檔案中插入 SpringDoc 設定

```properties
# SpringDoc 設定
springdoc.swagger-ui.doc-expansion=none
springdoc.swagger-ui.operations-sorter=alpha
springdoc.swagger-ui.tags-sorter=alpha
```

- 在 `application.properties` 檔案中插入 Redis 設定

```properties
# Redis 設定
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.data.redis.password=rootroot
```

- 在 `application.properties` 檔案中插入 R2DBC 設定

```properties
# R2DBC 設定
spring.r2dbc.url=r2dbc:postgresql://localhost:5432/postgres
spring.r2dbc.username=postgres
spring.r2dbc.password=rootroot

spring.sql.init.mode=always
spring.sql.init.platform=postgres
spring.sql.init.continue-on-error=true
```

- 在 `application.properties` 檔案中插入 MongoDB 設定

```properties
# MongoDB 設定
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.username=root
spring.data.mongodb.password=rootroot
spring.data.mongodb.database=test
```

- 在專案根目錄建立 `docker-compose.yaml`，並新增下列服務：`redis:6`、`postgresql:17` 和 `mongo:8`。

  - redis 服務應該具備
    - 密碼為 `rootroot`
    - 將連接埠 6379 對應到 6379
    - 將磁碟區 `./redis_data` 掛載到 `/data`
  - postgresql 服務應該具備
    - 密碼為 `rootroot`
    - 將連接埠 5432 對應到 5432
    - 將磁碟區 `./postgres_data` 掛載到 `/var/lib/postgresql/data`
  - mongo 服務應該具備
    - 初始化資料庫根用戶名稱 `root`
    - 初始化資料庫根密碼 `rootroot`
    - 將連接埠 27017 對應到 27017
    - 將磁碟區 `./mongo_data` 掛載到 `/data/db`

- 在 `.gitignore` 檔案中插入 `redis_data`、`postgres_data` 和 `mongo_data` 目錄

- 執行 gradle clean test 命令檢查專案是否正常運作

```shell
./gradlew clean test
```

- （選擇性）執行 `docker-compose up -d` 啟動服務、執行 `./gradlew spring-boot:run` 執行 Spring Boot 專案、執行 `docker-compose rm -sf` 停止服務。

讓我們逐步進行。

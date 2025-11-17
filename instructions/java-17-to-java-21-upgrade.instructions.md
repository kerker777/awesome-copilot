---
applyTo: ['*']
description: "Comprehensive best practices for adopting new Java 21 features since the release of Java 17."
---

# Java 17 升級至 Java 21 指南

這些指示可協助 GitHub Copilot 協助開發人員將 Java 專案從 JDK 17 升級至 JDK 21，重點著重於新語言功能、API 變更及最佳實踐。

## JDK 18-21 主要語言功能

### switch 的模式匹配 (JEP 441 - Java 21 標準功能)

**增強的 switch 表達式和陳述句**

使用 switch 結構時：
- 在適當的地方建議將傳統 switch 轉換為模式匹配
- 使用模式匹配進行型別檢查和解構
- 升級模式範例：
```java
// Old approach (Java 17)
public String processObject(Object obj) {
    if (obj instanceof String) {
        String s = (String) obj;
        return s.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return i.toString();
    }
    return "unknown";
}

// New approach (Java 21)
public String processObject(Object obj) {
    return switch (obj) {
        case String s -> s.toUpperCase();
        case Integer i -> i.toString();
        case null -> "null";
        default -> "unknown";
    };
}
```

- 支援受保護的模式：
```java
switch (obj) {
    case String s when s.length() > 10 -> "Long string: " + s;
    case String s -> "Short string: " + s;
    case Integer i when i > 100 -> "Large number: " + i;
    case Integer i -> "Small number: " + i;
    default -> "Other";
}
```

### 記錄模式 (JEP 440 - Java 21 標準功能)

**在模式匹配中解構記錄**

使用記錄時：
- 建議使用記錄模式進行解構
- 與 switch 表達式結合以實現強大的資料處理
- 使用範例：
```java
public record Point(int x, int y) {}
public record ColoredPoint(Point point, Color color) {}

// Destructuring in switch
public String describe(Object obj) {
    return switch (obj) {
        case Point(var x, var y) -> "Point at (" + x + ", " + y + ")";
        case ColoredPoint(Point(var x, var y), var color) -> 
            "Colored point at (" + x + ", " + y + ") in " + color;
        default -> "Unknown shape";
    };
}
```

- 在複雜模式匹配中使用：
```java
// Nested record patterns
switch (shape) {
    case Rectangle(ColoredPoint(Point(var x1, var y1), var c1), 
                   ColoredPoint(Point(var x2, var y2), var c2)) 
        when c1 == c2 -> "Monochrome rectangle";
    case Rectangle r -> "Multi-colored rectangle";
}
```

### 虛擬執行緒 (JEP 444 - Java 21 標準功能)

**輕量級並行處理**

使用並行處理時：
- 建議在高吞吐量的並行應用程式中使用虛擬執行緒
- 使用 `Thread.ofVirtual()` 建立虛擬執行緒
- 遷移模式範例：
```java
// Old platform thread approach
ExecutorService executor = Executors.newFixedThreadPool(100);
executor.submit(() -> {
    // blocking I/O operation
    httpClient.send(request);
});

// New virtual thread approach
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> {
        // blocking I/O operation - now scales to millions
        httpClient.send(request);
    });
}
```

- 使用結構化並行模式：
```java
// Structured concurrency (Preview)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser(userId));
    Future<String> order = scope.fork(() -> fetchOrder(orderId));
    
    scope.join();           // Join all subtasks
    scope.throwIfFailed();  // Propagate errors
    
    return processResults(user.resultNow(), order.resultNow());
}
```

### 字串模板 (JEP 430 - Java 21 預覽功能)

**安全的字串插補**

使用字串格式化時：
- 建議使用字串模板進行安全的字串插補（預覽功能）
- 使用 `--enable-preview` 啟用預覽功能
- 使用範例：
```java
// Traditional concatenation
String message = "Hello, " + name + "! You have " + count + " messages.";

// String Templates (Preview)
String message = STR."Hello, \{name}! You have \{count} messages.";

// Safe HTML generation
String html = HTML."<p>User: \{username}</p>";

// Safe SQL queries  
PreparedStatement stmt = SQL."SELECT * FROM users WHERE id = \{userId}";
```

### 有序集合 (JEP 431 - Java 21 標準功能)

**增強的集合介面**

使用集合時：
- 使用新的 `SequencedCollection`、`SequencedSet`、`SequencedMap` 介面
- 在不同集合型別間統一地存取第一個和最後一個元素
- 使用範例：
```java
// New methods available on Lists, Deques, LinkedHashSet, etc.
List<String> list = List.of("first", "middle", "last");
String first = list.getFirst();  // "first"
String last = list.getLast();    // "last"
List<String> reversed = list.reversed(); // ["last", "middle", "first"]

// Works with any SequencedCollection
SequencedSet<String> set = new LinkedHashSet<>();
set.addFirst("start");
set.addLast("end");
String firstElement = set.getFirst();
```

### 未命名模式和變數 (JEP 443 - Java 21 預覽功能)

**簡化的模式匹配**

使用模式匹配時：
- 對於不需要的值使用未命名模式 `_`
- 簡化 switch 表達式和記錄模式
- 使用範例：
```java
// Ignore unused variables
switch (ball) {
    case RedBall(_) -> "Red ball";     // Don't care about size
    case BlueBall(var size) -> "Blue ball size " + size;
}

// Ignore parts of records
switch (point) {
    case Point(var x, _) -> "X coordinate: " + x; // Ignore Y
    case ColoredPoint(Point(_, var y), _) -> "Y coordinate: " + y;
}

// Exception handling with unnamed variables
try {
    riskyOperation();
} catch (IOException | SQLException _) {
    // Don't need exception details
    handleError();
}
```

### 有限範圍的值 (JEP 446 - Java 21 預覽功能)

**改進的上下文傳播**

使用執行緒本地資料時：
- 考慮使用有限範圍的值作為 ThreadLocal 的現代替代方案
- 對虛擬執行緒提供更好的效能和更清晰的語義
- 使用範例：
```java
// Define scoped value
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

// Set and use scoped value
ScopedValue.where(USER_ID, "user123")
    .run(() -> {
        processRequest(); // Can access USER_ID.get() anywhere in call chain
    });

// In nested method
public void processRequest() {
    String userId = USER_ID.get(); // "user123"
    // Process with user context
}
```

## API 增強和新功能

### 預設 UTF-8 (JEP 400 - Java 18 標準功能)

使用檔案 I/O 時：
- UTF-8 現在是所有平台上的預設字元集
- 移除預期使用 UTF-8 的明確字元集規格
- 簡化範例：
```java
// Old explicit UTF-8 specification
Files.readString(path, StandardCharsets.UTF_8);
Files.writeString(path, content, StandardCharsets.UTF_8);

// New default behavior (Java 18+)
Files.readString(path);  // Uses UTF-8 by default
Files.writeString(path, content);  // Uses UTF-8 by default
```

### 簡單網頁伺服器 (JEP 408 - Java 18 標準功能)

需要基本 HTTP 伺服器時：
- 使用內建的 `jwebserver` 命令或 `com.sun.net.httpserver` 增強
- 適合測試和開發
- 使用範例：
```java
// Command line
$ jwebserver -p 8080 -d /path/to/files

// Programmatic usage
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
server.createContext("/", new SimpleFileHandler(Path.of("/tmp")));
server.start();
```

### 網際網路位址解析 SPI (JEP 418 - Java 19 標準功能)

使用自訂 DNS 解析時：
- 實作 `InetAddressResolverProvider` 進行自訂位址解析
- 適用於服務發現和測試案例

### 金鑰封裝機制 API (JEP 452 - Java 21 標準功能)

使用後量子密碼學時：
- 使用 KEM API 進行金鑰封裝機制
- 使用範例：
```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("ML-KEM");
KeyPair kp = kpg.generateKeyPair();

KEM kem = KEM.getInstance("ML-KEM");
KEM.Encapsulator encapsulator = kem.newEncapsulator(kp.getPublic());
KEM.Encapsulated encapsulated = encapsulator.encapsulate();
```

## 棄用和警告

### 終止化棄用 (JEP 421 - Java 18 棄用)

遇到 `finalize()` 方法時：
- 移除 finalize 方法並使用替代方案
- 建議使用 Cleaner API 或 try-with-resources
- 遷移範例：
```java
// Deprecated finalize approach
@Override
protected void finalize() throws Throwable {
    cleanup();
}

// Modern approach with Cleaner
private static final Cleaner CLEANER = Cleaner.create();

public MyResource() {
    cleaner.register(this, new CleanupTask(nativeResource));
}

private static class CleanupTask implements Runnable {
    private final long nativeResource;
    
    CleanupTask(long nativeResource) {
        this.nativeResource = nativeResource;
    }
    
    public void run() {
        cleanup(nativeResource);
    }
}
```

### 動態代理載入 (JEP 451 - Java 21 警告)

使用代理或工具時：
- 在需要時新增 `-XX:+EnableDynamicAgentLoading` 以抑制警告
- 考慮在啟動時載入代理而非動態載入
- 更新工具以使用啟動時代理載入

## 建置設定更新

### 預覽功能

對於使用預覽功能的專案：
- 在編譯器和執行時新增 `--enable-preview`
- Maven 設定：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <release>21</release>
        <compilerArgs>
            <arg>--enable-preview</arg>
        </compilerArgs>
    </configuration>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>--enable-preview</argLine>
    </configuration>
</plugin>
```

- Gradle 設定：
```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

tasks.withType<JavaCompile> {
    options.compilerArgs.add("--enable-preview")
}

tasks.withType<Test> {
    jvmArgs("--enable-preview")
}
```

### 虛擬執行緒設定

對於使用虛擬執行緒的應用程式：
- 不需要特殊的 JVM 旗標（Java 21 中的標準功能）
- 考慮這些系統屬性用於除錯：
```bash
-Djdk.virtualThreadScheduler.parallelism=N  # Set carrier thread count
-Djdk.virtualThreadScheduler.maxPoolSize=N  # Set max pool size
```

## 執行時和垃圾回收改進

### 世代制 ZGC (JEP 439 - Java 21 可用)

設定垃圾回收時：
- 嘗試世代制 ZGC 以改進效能
- 使用以下方式啟用：`-XX:+UseZGC -XX:+ZGenerational`
- 監控配置模式和垃圾回收行為

## 遷移策略

### 逐步升級程序

1. **更新建置工具**：確保 Maven/Gradle 支援 JDK 21
2. **語言功能採用**：
   - 從 switch 的模式匹配開始（標準功能）
   - 在有益處的地方新增記錄模式
   - 針對 I/O 密集型應用程式考慮使用虛擬執行緒
3. **預覽功能**：只在特定用途需要時啟用
4. **測試**：全面測試，特別是並行處理的變更
5. **效能**：使用新的垃圾回收選項進行基準測試

### 程式碼審查檢查清單

審查 Java 21 升級的程式碼時：
- [ ] 將適當的 instanceof 鏈轉換為 switch 表達式
- [ ] 使用記錄模式進行資料解構
- [ ] 在適當的地方將 ThreadLocal 替換為 ScopedValues
- [ ] 針對高並行案例考慮使用虛擬執行緒
- [ ] 移除明確的 UTF-8 字元集規格
- [ ] 使用 Cleaner 或 try-with-resources 替換 finalize() 方法
- [ ] 使用 SequencedCollection 方法進行第一個/最後一個元素存取模式
- [ ] 只為使用中的預覽功能新增預覽旗標

### 常見遷移模式

1. **Switch 增強**：
   ```java
   // From instanceof chains to switch expressions
   if (obj instanceof String s) return processString(s);
   else if (obj instanceof Integer i) return processInt(i);
   // becomes:
   return switch (obj) {
       case String s -> processString(s);
       case Integer i -> processInt(i);
       default -> processDefault(obj);
   };
   ```

2. **虛擬執行緒採用**：
   ```java
   // From platform threads to virtual threads
   Executors.newFixedThreadPool(200)
   // becomes:
   Executors.newVirtualThreadPerTaskExecutor()
   ```

3. **記錄模式使用**：
   ```java
   // From manual destructuring to record patterns
   if (point instanceof Point p) {
       int x = p.x();
       int y = p.y();
   }
   // becomes:
   if (point instanceof Point(var x, var y)) {
       // use x and y directly
   }
   ```

## 效能考量

- 虛擬執行緒在阻止 I/O 時表現出色，但可能不利於 CPU 密集型任務
- 世代制 ZGC 可以減少大多數應用程式的垃圾回收開銷
- switch 中的模式匹配通常比 instanceof 鏈更有效率
- SequencedCollection 方法提供對第一個/最後一個元素的 O(1) 存取
- Scoped Values 對虛擬執行緒的開銷比 ThreadLocal 低

## 測試建議

- 在高並行狀況下測試虛擬執行緒應用程式
- 驗證模式匹配涵蓋所有預期的案例
- 進行世代制 ZGC 與其他回收器的效能測試
- 驗證 UTF-8 預設行為在不同平台上的表現
- 在生產環境使用前徹底測試預覽功能

請記得只在特別需要時啟用預覽功能，並在暫存環境中徹底測試後再部署到生產環境。

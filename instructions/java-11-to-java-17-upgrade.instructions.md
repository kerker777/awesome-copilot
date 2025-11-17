---
applyTo: ["*"]
description: "Comprehensive best practices for adopting new Java 17 features since the release of Java 11."
---

# Java 11 至 Java 17 升級指南

## 專案背景

本指南提供全面的 GitHub Copilot 指示，用於升級 Java 專案從 JDK 11 至 JDK 17，涵蓋在這兩個版本間整合的 47 項 JEP 中的主要語言功能、API 變更和遷移模式。

## 語言功能與 API 變更

### JEP 395：記錄類型（Java 16）

**遷移模式**：將資料類別轉換為記錄類型

```java
// Old: Traditional data class
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() { return name; }
    public int age() { return age; }

    @Override
    public boolean equals(Object obj) { /* boilerplate */ }
    @Override
    public int hashCode() { /* boilerplate */ }
    @Override
    public String toString() { /* boilerplate */ }
}

// New: Record (Java 16+)
public record Person(String name, int age) {
    // Compact constructor for validation
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }

    // Custom methods can be added
    public boolean isAdult() {
        return age >= 18;
    }
}
```

### JEP 409：密封類別（Java 17）

**遷移模式**：使用密封類別來限制繼承

```java
// New: Sealed class hierarchy
public sealed class Shape
    permits Circle, Rectangle, Triangle {

    public abstract double area();
}

public final class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public final class Rectangle extends Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}

public non-sealed class Triangle extends Shape {
    // Non-sealed allows further inheritance
    private final double base, height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double area() {
        return 0.5 * base * height;
    }
}
```

### JEP 394：instanceof 的型態匹配（Java 16）

**遷移模式**：簡化 instanceof 檢查

```java
// Old: Traditional instanceof with casting
public String processObject(Object obj) {
    if (obj instanceof String) {
        String str = (String) obj;
        return str.toUpperCase();
    } else if (obj instanceof Integer) {
        Integer num = (Integer) obj;
        return "Number: " + num;
    } else if (obj instanceof List<?>) {
        List<?> list = (List<?>) obj;
        return "List with " + list.size() + " elements";
    }
    return "Unknown type";
}

// New: Pattern matching for instanceof (Java 16+)
public String processObject(Object obj) {
    if (obj instanceof String str) {
        return str.toUpperCase();
    } else if (obj instanceof Integer num) {
        return "Number: " + num;
    } else if (obj instanceof List<?> list) {
        return "List with " + list.size() + " elements";
    }
    return "Unknown type";
}

// Works great with sealed classes
public String describeShape(Shape shape) {
    if (shape instanceof Circle circle) {
        return "Circle with radius " + circle.radius();
    } else if (shape instanceof Rectangle rect) {
        return "Rectangle " + rect.width() + "x" + rect.height();
    } else if (shape instanceof Triangle triangle) {
        return "Triangle with base " + triangle.base();
    }
    return "Unknown shape";
}
```

### JEP 361：switch 表達式（Java 14）

**遷移模式**：將 switch 陳述句轉換為表達式

```java
// Old: Traditional switch statement
public String getDayType(DayOfWeek day) {
    String result;
    switch (day) {
        case MONDAY:
        case TUESDAY:
        case WEDNESDAY:
        case THURSDAY:
        case FRIDAY:
            result = "Workday";
            break;
        case SATURDAY:
        case SUNDAY:
            result = "Weekend";
            break;
        default:
            throw new IllegalArgumentException("Unknown day: " + day);
    }
    return result;
}

// New: Switch expression (Java 14+)
public String getDayType(DayOfWeek day) {
    return switch (day) {
        case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Workday";
        case SATURDAY, SUNDAY -> "Weekend";
    };
}

// With yield for complex logic
public int calculateScore(Grade grade) {
    return switch (grade) {
        case A -> 100;
        case B -> 85;
        case C -> 70;
        case D -> {
            System.out.println("Consider improvement");
            yield 55;
        }
        case F -> {
            System.out.println("Needs retake");
            yield 0;
        }
    };
}
```

### JEP 406：switch 的型態匹配（Java 17 預覽功能）

**遷移模式**：強化的 switch 與型態匹配（預覽功能）

```java
// Requires --enable-preview flag
public String formatValue(Object obj) {
    return switch (obj) {
        case String s -> "String: " + s;
        case Integer i -> "Integer: " + i;
        case null -> "null value";
        case default -> "Unknown: " + obj.getClass().getSimpleName();
    };
}

// With guarded patterns
public String categorizeNumber(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Negative integer";
        case Integer i when i == 0 -> "Zero";
        case Integer i when i > 0 -> "Positive integer";
        case Double d when d.isNaN() -> "Not a number";
        case Number n -> "Other number: " + n;
        case null -> "null";
        case default -> "Not a number";
    };
}
```

### JEP 378：文字區塊（Java 15）

**遷移模式**：使用文字區塊處理多行字串

```java
// Old: Concatenated strings
String html = "<html>\n" +
              "  <body>\n" +
              "    <h1>Hello World</h1>\n" +
              "    <p>Welcome to Java 17!</p>\n" +
              "  </body>\n" +
              "</html>";

String sql = "SELECT p.id, p.name, p.email, " +
             "       a.street, a.city, a.state " +
             "FROM person p " +
             "JOIN address a ON p.address_id = a.id " +
             "WHERE p.active = true " +
             "ORDER BY p.name";

// New: Text blocks (Java 15+)
String html = """
              <html>
                <body>
                  <h1>Hello World</h1>
                  <p>Welcome to Java 17!</p>
                </body>
              </html>
              """;

String sql = """
             SELECT p.id, p.name, p.email,
                    a.street, a.city, a.state
             FROM person p
             JOIN address a ON p.address_id = a.id
             WHERE p.active = true
             ORDER BY p.name
             """;

// With string interpolation methods
String json = """
              {
                "name": "%s",
                "age": %d,
                "city": "%s"
              }
              """.formatted(name, age, city);
```

### JEP 358：有幫助的 NullPointerExceptions（Java 14）

**遷移指導**：更好的 NPE 除錯（在 Java 17 中預設啟用）

```java
// Old NPE message: "Exception in thread 'main' java.lang.NullPointerException"
// New NPE message shows exactly what was null:
// "Cannot invoke 'String.length()' because the return value of 'Person.getName()' is null"

public class PersonProcessor {
    public void processPersons(List<Person> persons) {
        // This will show exactly which person.getName() returned null
        persons.stream()
            .mapToInt(person -> person.getName().length())  // Clear NPE if getName() returns null
            .sum();
    }

    // Better error messages help with complex expressions
    public void complexExample(Map<String, List<Person>> groups) {
        // NPE will show exactly which part of the chain is null
        int totalNameLength = groups.get("admins")
                                  .get(0)
                                  .getName()
                                  .length();
    }
}
```

### JEP 371：隱藏類別（Java 15）

**遷移模式**：用於框架和代理生成

```java
// For frameworks creating dynamic proxies
public class DynamicProxyExample {
    public static <T> T createProxy(Class<T> interfaceClass, InvocationHandler handler) {
        // Hidden classes provide better encapsulation for dynamically generated classes
        MethodHandles.Lookup lookup = MethodHandles.lookup();

        // Framework code would use hidden classes for better isolation
        // This is typically handled by frameworks, not application code
        return interfaceClass.cast(
            Proxy.newProxyInstance(
                interfaceClass.getClassLoader(),
                new Class<?>[]{interfaceClass},
                handler
            )
        );
    }
}
```

### JEP 334：JVM 常數 API（Java 12）

**遷移模式**：用於編譯時期常數

```java
import java.lang.constant.*;

// For advanced metaprogramming and tooling
public class ConstantExample {
    // Use dynamic constants for computed values
    public static final DynamicConstantDesc<String> COMPUTED_CONSTANT =
        DynamicConstantDesc.of(
            ConstantDescs.BSM_INVOKE,
            "computeValue",
            ConstantDescs.CD_String
        );

    // Primarily used by compiler and framework developers
    public static String computeValue() {
        return "Computed at runtime, cached as constant";
    }
}
```

### JEP 415：上下文特定的反序列化篩選器（Java 17）

**遷移模式**：加強物件反序列化的安全性

```java
import java.io.*;

public class SecureDeserialization {
    // Set up deserialization filters for security
    public static void setupSerializationFilters() {
        // Global filter
        ObjectInputFilter globalFilter = ObjectInputFilter.Config.createFilter(
            "java.base/*;java.util.*;!*"
        );
        ObjectInputFilter.Config.setSerialFilter(globalFilter);
    }

    public <T> T deserializeSecurely(byte[] data, Class<T> expectedType) throws IOException, ClassNotFoundException {
        try (ByteArrayInputStream bis = new ByteArrayInputStream(data);
             ObjectInputStream ois = new ObjectInputStream(bis)) {

            // Context-specific filter
            ObjectInputFilter contextFilter = ObjectInputFilter.Config.createFilter(
                expectedType.getName() + ";java.lang.*;!*"
            );
            ois.setObjectInputFilter(contextFilter);

            return expectedType.cast(ois.readObject());
        }
    }
}
```

### JEP 356：強化的虛擬亂數產生器（Java 17）

**遷移模式**：使用新的亂數產生器介面

```java
import java.util.random.*;

// Old: Limited Random class
Random oldRandom = new Random();
int oldValue = oldRandom.nextInt(100);

// New: Enhanced random generators (Java 17+)
RandomGenerator generator = RandomGeneratorFactory
    .of("Xoshiro256PlusPlus")
    .create(System.nanoTime());

RandomGenerator.SplittableGenerator splittableGenerator =
    RandomGeneratorFactory.of("L64X128MixRandom").create();

// Better for parallel processing
splittableGenerator.splits(4)
    .parallel()
    .mapToInt(rng -> rng.nextInt(1000))
    .forEach(System.out::println);

// Streamable random values
generator.ints(10, 1, 101)
    .forEach(System.out::println);
```

## I/O 與網路改進

### JEP 380：Unix 網域通訊端通道（Java 16）

**遷移模式**：使用 Unix 網域通訊端進行本機 IPC

```java
import java.net.UnixDomainSocketAddress;
import java.nio.channels.*;

// Old: TCP sockets for local communication
// ServerSocketChannel server = ServerSocketChannel.open();
// server.bind(new InetSocketAddress("localhost", 8080));

// New: Unix domain sockets (Java 16+)
public class UnixSocketExample {
    public void createUnixDomainServer() throws IOException {
        Path socketPath = Path.of("/tmp/my-app.socket");
        UnixDomainSocketAddress address = UnixDomainSocketAddress.of(socketPath);

        try (ServerSocketChannel server = ServerSocketChannel.open(StandardProtocolFamily.UNIX)) {
            server.bind(address);

            while (true) {
                try (SocketChannel client = server.accept()) {
                    // Handle client connection
                    handleClient(client);
                }
            }
        }
    }

    public void connectToUnixSocket() throws IOException {
        Path socketPath = Path.of("/tmp/my-app.socket");
        UnixDomainSocketAddress address = UnixDomainSocketAddress.of(socketPath);

        try (SocketChannel client = SocketChannel.open(address)) {
            // Communicate with server
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            client.read(buffer);
        }
    }

    private void handleClient(SocketChannel client) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        int bytesRead = client.read(buffer);
        // Process client data
    }
}
```

### JEP 352：非揮發性映射位元組緩衝區（Java 14）

**遷移模式**：用於持久性記憶體操作

```java
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.StandardOpenOption;

public class PersistentMemoryExample {
    public void usePersistentMemory() throws IOException {
        Path nvmFile = Path.of("/mnt/pmem/data.bin");

        try (FileChannel channel = FileChannel.open(nvmFile,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE,
                StandardOpenOption.CREATE)) {

            // Map as persistent memory
            MappedByteBuffer buffer = channel.map(
                FileChannel.MapMode.READ_WRITE, 0, 1024,
                ExtendedMapMode.READ_WRITE_SYNC
            );

            // Write data that persists across crashes
            buffer.putLong(0, System.currentTimeMillis());
            buffer.putInt(8, 12345);

            // Force write to persistent storage
            buffer.force();
        }
    }
}
```

## 建構系統組態

### Maven 組態

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <maven.compiler.release>17</maven.compiler.release>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <release>17</release>
                <!-- Enable preview features if using JEP 406 -->
                <compilerArgs>
                    <arg>--enable-preview</arg>
                </compilerArgs>
            </configuration>
        </plugin>

        <!-- For running tests with preview features -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <argLine>--enable-preview</argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Gradle 組態

```kotlin
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

tasks.withType<JavaCompile> {
    options.release.set(17)
    // Enable preview features if needed
    options.compilerArgs.addAll(listOf("--enable-preview"))
}

tasks.withType<Test> {
    useJUnitPlatform()
    // Enable preview features for tests
    jvmArgs("--enable-preview")
}
```

## 棄用與移除

### JEP 411：棄用安全管理器以進行移除

**遷移模式**：移除安全管理器的依賴

```java
// Old: Using Security Manager
SecurityManager sm = System.getSecurityManager();
if (sm != null) {
    sm.checkPermission(new RuntimePermission("shutdownHooks"));
}

// New: Alternative security approaches
// Use application-level security, containers, or process isolation
// Most applications don't need Security Manager functionality
```

### JEP 398：棄用小程式 API 以進行移除

**遷移模式**：從小程式遷移至現代網路技術

```java
// Old: Java Applet (deprecated)
public class MyApplet extends Applet {
    @Override
    public void start() {
        // Applet code
    }
}

// New: Modern alternatives
// 1. Convert to standalone Java application
public class MyApplication extends JFrame {
    public MyApplication() {
        setTitle("My Application");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        // Application code
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new MyApplication().setVisible(true);
        });
    }
}

// 2. Use Java Web Start alternative (jlink)
// 3. Convert to web application using modern frameworks
```

### JEP 372：移除 Nashorn JavaScript 引擎

**遷移模式**：使用替代的 JavaScript 引擎

```java
// Old: Nashorn (removed in Java 17)
// ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

// New: Alternative approaches
// 1. Use GraalVM JavaScript engine
ScriptEngine engine = new ScriptEngineManager().getEngineByName("graal.js");

// 2. Use external JavaScript execution
ProcessBuilder pb = new ProcessBuilder("node", "script.js");
Process process = pb.start();

// 3. Use web-based approach or embedded browser
```

## JVM 與效能改進

### JEP 377：ZGC - 可擴展的低延遲垃圾收集器（Java 15）

**遷移模式**：為低延遲應用程式啟用 ZGC

```bash
# Enable ZGC
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions  # Not needed in Java 17

# Monitor ZGC performance
-XX:+LogVMOutput
-XX:LogFile=gc.log
```

### JEP 379：Shenandoah - 低暫停時間垃圾收集器（Java 15）

**遷移模式**：啟用 Shenandoah 以獲得一致的延遲

```bash
# Enable Shenandoah
-XX:+UseShenandoahGC
-XX:+UnlockExperimentalVMOptions  # Not needed in Java 17

# Shenandoah tuning
-XX:ShenandoahGCHeuristics=adaptive
```

### JEP 341：預設 CDS 檔案（Java 12）與 JEP 350：動態 CDS 檔案（Java 13）

**遷移模式**：改進啟動效能

```bash
# CDS is enabled by default, but you can create custom archives
# Create custom CDS archive
java -XX:DumpLoadedClassList=classes.lst -cp myapp.jar com.example.Main
java -Xshare:dump -XX:SharedClassListFile=classes.lst -XX:SharedArchiveFile=myapp.jsa -cp myapp.jar

# Use custom CDS archive
java -XX:SharedArchiveFile=myapp.jsa -cp myapp.jar com.example.Main
```

## 測試與遷移策略

### 第 1 階段：基礎（第 1-2 週）

1. **更新建構系統**

   - 修改 Maven/Gradle 組態以支援 Java 17
   - 更新 CI/CD 管道
   - 驗證依賴相容性

2. **處理移除和棄用**
   - 移除 Nashorn JavaScript 引擎的使用
   - 取代已棄用的小程式 API
   - 更新安全管理器的使用

### 第 2 階段：語言功能（第 3-4 週）

1. **實作記錄類型**

   - 將資料類別轉換為記錄類型
   - 在緊縮建構函式中新增驗證
   - 測試序列化相容性

2. **新增型態匹配**
   - 轉換 instanceof 連鎖
   - 實作型別安全的轉型模式

### 第 3 階段：進階功能（第 5-6 週）

1. **Switch 表達式**

   - 將 switch 陳述句轉換為表達式
   - 使用新的箭頭語法
   - 實作複雜的 yield 邏輯

2. **文字區塊**
   - 取代連接的多行字串
   - 更新 SQL 和 HTML 生成
   - 使用格式化方法

### 第 4 階段：密封類別（第 7-8 週）

1. **設計密封類別層級結構**

   - 識別繼承限制
   - 實作密封類別模式
   - 與型態匹配結合

2. **測試與驗證**
   - 全面的測試覆蓋
   - 效能基準測試
   - 相容性驗證

## 效能考量

### 記錄類型與傳統類別

- 記錄類型的記憶體效率更高
- 建立和相等檢查速度更快
- 自動序列化支援
- 適合用於資料傳輸物件

### 型態匹配效能

- 消除多餘的型別檢查
- 減少轉型開銷
- 為 JVM 優化提供更多機會
- 與密封類別搭配以確保完整性

### Switch 表達式優化

- 產生更有效率的位元碼
- 更好的常數折疊
- 改進的分支預測
- 用於複雜的條件邏輯

## 最佳實踐

1. **為資料類別使用記錄類型**

   - 不可變資料容器
   - API 資料傳輸物件
   - 組態物件

2. **戰略性地應用型態匹配**

   - 取代 instanceof 連鎖
   - 與密封類別搭配
   - 與 switch 表達式結合

3. **採用文字區塊處理多行內容**

   - SQL 查詢
   - JSON 範本
   - HTML 內容
   - 組態檔案

4. **以密封類別進行設計**

   - 領域建模
   - 狀態機
   - 代數資料型別
   - API 演化控制

5. **善用強化的亂數產生器**
   - 平行處理情境
   - 高品質亂數
   - 統計應用
   - 遊戲與模擬

本全面指南使 GitHub Copilot 在升級 Java 11 專案至 Java 17 時，能夠提供適當的語境建議，重點關注語言增強、API 改進和現代 Java 開發實踐。

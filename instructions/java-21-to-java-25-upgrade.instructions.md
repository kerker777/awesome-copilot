---
applyTo: ['*']
description: "Comprehensive best practices for adopting new Java 25 features since the release of Java 21."
---

# Java 21 至 Java 25 升級指南

這些指示幫助 GitHub Copilot 協助開發人員將 Java 專案從 JDK 21 升級到 JDK 25，重點放在新語言特性、API 變更和最佳實踐。

## JDK 22-25 的語言特性和 API 變更

### 模式匹配增強（JEP 455/488 - 23 中預覽）

**模式中的原始類型、instanceof 和 switch**

在處理模式匹配時：
- 建議在 switch 運算式和 instanceof 檢查中使用原始類型模式
- 從傳統 switch 升級的範例：
```java
// 舊方法 (Java 21)
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    default -> "unknown status: " + x.getStatus();
}

// 新方法 (Java 25 預覽)
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    case int i -> "unknown status: " + i;
}
```

- 使用 `--enable-preview` 旗標啟用預覽功能
- 建議針對更複雜的條件使用防衛模式：
```java
switch (x.getYearlyFlights()) {
    case 0 -> ...;
    case int i when i >= 100 -> issueGoldCard();
    case int i -> ... // 處理 1-99 範圍
}
```

### 類別檔案 API（JEP 466/484 - 23 中第二個預覽，25 中標準）

**將 ASM 替換為標準 API**

當偵測到位元組碼操作或類別檔案處理時：
- 建議從 ASM 函式庫遷移到標準類別檔案 API
- 使用 `java.lang.classfile` 套件而不是 `org.objectweb.asm`
- 遷移模式範例：
```java
// 舊 ASM 方法
ClassReader reader = new ClassReader(classBytes);
ClassWriter writer = new ClassWriter(reader, 0);
// ... ASM 操作

// 新類別檔案 API 方法
ClassModel classModel = ClassFile.of().parse(classBytes);
byte[] newBytes = ClassFile.of().transform(classModel,
    ClassTransform.transformingMethods(methodTransform));
```

### Markdown 文件註解（JEP 467 - 23 中標準）

**JavaDoc 現代化**

在處理 JavaDoc 註解時：
- 建議將 HTML 密集的 JavaDoc 轉換為 Markdown 語法
- 使用 `///` 進行 Markdown 文件註解
- 轉換範例：
```java
// 舊 HTML JavaDoc
/**
 * Returns the <b>absolute</b> value of an {@code int} value.
 * <p>
 * If the argument is not negative, return the argument.
 * If the argument is negative, return the negation of the argument.
 *
 * @param a the argument whose absolute value is to be determined
 * @return the absolute value of the argument
 */

// 新 Markdown JavaDoc
/// Returns the **absolute** value of an `int` value.
///
/// If the argument is not negative, return the argument.
/// If the argument is negative, return the negation of the argument.
///
/// @param a the argument whose absolute value is to be determined
/// @return the absolute value of the argument
```

### 衍生記錄建立（JEP 468 - 23 中預覽）

**記錄增強**

在處理記錄時：
- 建議使用 `with` 運算式來建立衍生記錄
- 為衍生記錄建立啟用預覽功能
- 範例模式：
```java
// 不再手動複製記錄
public record Person(String name, int age, String email) {
    public Person withAge(int newAge) {
        return new Person(name, newAge, email);
    }
}

// 使用衍生記錄建立（預覽）
Person updated = person with { age = 30; };
```

### 流收集器（JEP 473/485 - 23 中第二個預覽，25 中標準）

**增強的串流處理**

在處理複雜的串流操作時：
- 建議使用 `Stream.gather()` 進行自訂中間操作
- 為內建的收集器匯入 `java.util.stream.Gatherers`
- 使用範例：
```java
// 自訂視窗操作
List<List<String>> windows = stream
    .gather(Gatherers.windowSliding(3))
    .toList();

// 使用狀態進行自訂篩選
List<Integer> filtered = numbers.stream()
    .gather(Gatherers.fold(0, (state, element) -> {
        // 自訂有狀態邏輯
        return state + element > threshold ? element : null;
    }))
    .filter(Objects::nonNull)
    .toList();
```

## 遷移警告和棄用

### sun.misc.Unsafe 記憶體存取方法（JEP 471 - 23 中棄用）

偵測到 `sun.misc.Unsafe` 使用時：
- 警告有關棄用的記憶體存取方法
- 建議遷移到標準替代方案：
```java
// 棄用：sun.misc.Unsafe 記憶體存取
Unsafe unsafe = Unsafe.getUnsafe();
unsafe.getInt(object, offset);

// 首選：VarHandle API
VarHandle vh = MethodHandles.lookup()
    .findVarHandle(MyClass.class, "fieldName", int.class);
int value = (int) vh.get(object);

// 或針對堆外記憶體：外部函式與記憶體 API
MemorySegment segment = MemorySegment.ofArray(new int[10]);
int value = segment.get(ValueLayout.JAVA_INT, offset);
```

### JNI 使用警告（JEP 472 - 24 中警告）

偵測到 JNI 使用時：
- 警告有關 JNI 使用的即將來臨的限制
- 建議為使用 JNI 的應用程式新增 `--enable-native-access` 旗標
- 建議盡可能遷移到外部函式與記憶體 API
- 為原生存取新增 module-info.java 項目：
```java
module com.example.app {
    requires jdk.unsupported; // 用於剩餘 JNI 使用
}
```

## 垃圾回收更新

### ZGC 代際模式（JEP 474 - 23 中預設）

在設定垃圾回收時：
- 預設 ZGC 現在使用代際模式
- 如果明確使用非代際 ZGC，請更新 JVM 旗標：
```bash
# 明確非代際模式（將顯示棄用警告）
-XX:+UseZGC -XX:-ZGenerational

# 預設代際模式
-XX:+UseZGC
```

### G1 改進（JEP 475 - 24 中實施）

使用 G1GC 時：
- 不需要程式碼變更 - 內部 JVM 最佳化
- 可能看到 C2 編譯器的編譯效能改進

## 向量 API（JEP 469 - 25 中第八個孵化器）

在處理數值計算時：
- 建議將向量 API 用於 SIMD 操作（仍在孵化中）
- 新增 `--add-modules jdk.incubator.vector`
- 使用範例：
```java
import jdk.incubator.vector.*;

// 傳統純量計算
for (int i = 0; i < a.length; i++) {
    c[i] = a[i] + b[i];
}

// 向量化計算
var species = IntVector.SPECIES_PREFERRED;
for (int i = 0; i < a.length; i += species.length()) {
    var va = IntVector.fromArray(species, a, i);
    var vb = IntVector.fromArray(species, b, i);
    var vc = va.add(vb);
    vc.intoArray(c, i);
}
```

## 編譯和建置設定

### 預覽功能

對於使用預覽功能的專案：
- 將 `--enable-preview` 新增至編譯器引數
- 將 `--enable-preview` 新增至執行階段引數
- Maven 設定：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <release>25</release>
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
        languageVersion = JavaLanguageVersion.of(25)
    }
}

tasks.withType<JavaCompile> {
    options.compilerArgs.add("--enable-preview")
}

tasks.withType<Test> {
    jvmArgs("--enable-preview")
}
```

## 遷移策略

### 逐步升級流程

1. **更新建置工具**：確保 Maven/Gradle 支援 JDK 25
2. **更新依賴套件**：檢查 JDK 25 相容性
3. **處理警告**：處理來自 JEP 471/472 的棄用警告
4. **啟用預覽功能**：如果使用模式匹配或其他預覽功能
5. **徹底測試**：特別是對於使用 JNI 或 sun.misc.Unsafe 的應用程式
6. **效能測試**：使用新的 ZGC 預設值驗證垃圾回收行為

### 程式碼審查清單

在審查 Java 25 升級的程式碼時：
- [ ] 將 ASM 使用取代為類別檔案 API
- [ ] 將複雜的 HTML JavaDoc 轉換為 Markdown
- [ ] 在適用的地方在 switch 運算式中使用原始類型模式
- [ ] 將 sun.misc.Unsafe 替換為 VarHandle 或 FFM API
- [ ] 為 JNI 使用新增原生存取權限
- [ ] 針對複雜的串流操作使用流收集器
- [ ] 更新建置設定以支援預覽功能

### 測試考量

- 使用 `--enable-preview` 旗標測試預覽功能
- 驗證 JNI 應用程式能與原生存取警告配合運作
- 使用新的 ZGC 代際模式進行效能測試
- 驗證使用 Markdown 註解的 JavaDoc 生成

## 常見陷阱

1. **預覽功能依賴**：不要在函式庫程式碼中使用預覽功能，除非有清楚的文件說明
2. **原生存取**：直接或間接使用 JNI 的應用程式可能需要 `--enable-native-access` 設定
3. **Unsafe 遷移**：不要延遲從 sun.misc.Unsafe 遷移 - 棄用警告表示未來會移除
4. **模式匹配範圍**：原始類型模式適用於所有原始類型，不只是 int
5. **記錄增強**：衍生記錄建立在 Java 23 中需要預覽旗標

## 效能考量

- ZGC 代際模式可能改進大多數工作負載的效能
- 類別檔案 API 減少 ASM 相關的負擔
- 流收集器為複雜的串流操作提供更好的效能
- G1GC 改進減少 JIT 編譯負擔

記得在將 Java 25 升級部署到生產系統之前，先在預備環境中徹底測試。

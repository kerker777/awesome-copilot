---
applyTo: '*'
description: 'Comprehensive best practices for deploying and managing applications on Kubernetes. Covers Pods, Deployments, Services, Ingress, ConfigMaps, Secrets, health checks, resource limits, scaling, and security contexts.'
---

# Kubernetes 部署最佳實踐

## 你的任務

作為 GitHub Copilot，你是 Kubernetes 部署的專家，具備深厚的知識，了解如何可靠、安全且高效地大規模運行應用程式的最佳實踐。你的任務是指導開發人員製作最優的 Kubernetes 宣告，管理部署，並確保他們的應用程式在 Kubernetes 環境中生產就緒。你必須強調復原力、安全性和可擴展性。

## Kubernetes 部署的核心概念

### **1. Pod**
- **原則：** Kubernetes 中最小的可部署單位。代表集群中單一執行程式的實例。
- **Copilot 指引：**
    - 設計 Pod 以執行單一主要容器（或緊密耦合的 sidecar）。
    - 定義 CPU 和記憶體的 `resources`（requests/limits）以防止資源耗盡。
    - 實作 `livenessProbe` 和 `readinessProbe` 進行健康檢查。
- **專業提示：** 避免直接部署 Pod；改用 Deployment 或 StatefulSet 等更高層級的控制器。

### **2. Deployment**
- **原則：** 管理一組相同的 Pod 並確保它們運行中。處理滾動式更新和回滾。
- **Copilot 指引：**
    - 為無狀態應用程式使用 Deployment。
    - 定義所需的副本數量（`replicas`）。
    - 指定 `selector` 和 `template` 以進行 Pod 匹配。
    - 為滾動式更新設定 `strategy`（`rollingUpdate`，包含 `maxSurge`/`maxUnavailable`）。
- **範例（簡單 Deployment）：**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: my-repo/my-app:1.0.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /readyz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

### **3. Service**
- **原則：** 一種將執行在一組 Pod 上的應用程式公開為網路服務的抽象方式。
- **Copilot 指引：**
    - 使用 Service 為 Pod 提供穩定的網路身份。
    - 根據公開需求選擇 `type`（ClusterIP、NodePort、LoadBalancer、ExternalName）。
    - 確保 `selector` 符合 Pod 標籤以進行正確的路由。
- **專業提示：** 內部服務使用 `ClusterIP`，雲端環境中面向網際網路的應用程式使用 `LoadBalancer`。

### **4. Ingress**
- **原則：** 管理對集群中服務的外部存取，通常是從集群外部到集群內服務的 HTTP/HTTPS 路由。
- **Copilot 指引：**
    - 使用 Ingress 整合路由規則並管理 TLS 終止。
    - 使用網路應用程式時，為外部存取設定 Ingress 資源。
    - 指定主機、路徑和後端服務。
- **範例（Ingress）：**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
  tls:
    - hosts:
        - myapp.example.com
      secretName: my-app-tls-secret
```

## 配置和密鑰管理

### **1. ConfigMap**
- **原則：** 將非敏感配置資料儲存為鍵值對。
- **Copilot 指引：**
    - 為應用程式配置、環境變數或命令列引數使用 ConfigMap。
    - 在 Pod 中將 ConfigMap 作為檔案掛載，或作為環境變數注入。
- **警告：** ConfigMap 在保存時未加密。不要在此儲存敏感資料。

### **2. Secret**
- **原則：** 安全地儲存敏感資料。
- **Copilot 指引：**
    - 為 API 金鑰、密碼、資料庫憑證、TLS 憑證使用 Kubernetes Secret。
    - 在 etcd 中以加密形式儲存密鑰（如果集群已設定）。
    - 將 Secret 掛載為卷（檔案）或作為環境變數注入（使用環境變數時謹慎）。
- **專業提示：** 生產環境中，使用外部密鑰操作者（例如 External Secrets Operator）與外部密鑰管理器（例如 HashiCorp Vault、AWS Secrets Manager、Azure Key Vault）整合。

## 健康檢查和探針

### **1. 活性探針（Liveness Probe）**
- **原則：** 判定容器是否仍在執行。如果失敗，Kubernetes 會重新啟動容器。
- **Copilot 指引：** 實作 HTTP、TCP 或命令式的活性探針，以確保應用程式處於活動狀態。
- **設定：** `initialDelaySeconds`、`periodSeconds`、`timeoutSeconds`、`failureThreshold`、`successThreshold`。

### **2. 就緒探針（Readiness Probe）**
- **原則：** 判定容器是否準備好處理流量。如果失敗，Kubernetes 會從 Service 負載均衡器中移除 Pod。
- **Copilot 指引：** 實作 HTTP、TCP 或命令式的就緒探針，以確保應用程式已完全初始化且相依服務可用。
- **專業提示：** 使用就緒探針在啟動或暫時中斷期間優雅地移除 Pod。

## 資源管理

### **1. 資源請求和限制（Resource Requests and Limits）**
- **原則：** 為每個容器定義 CPU 和記憶體的 requests/limits。
- **Copilot 指引：**
    - **Requests：** 保證的最少資源（用於排程）。
    - **Limits：** 硬性限制資源（防止有噪音的鄰居和資源耗盡）。
    - 建議同時設定 requests 和 limits 以確保服務品質（QoS）。
- **QoS 等級：** 了解 `Guaranteed`、`Burstable` 和 `BestEffort`。

### **2. 水平 Pod 自動調整器（Horizontal Pod Autoscaler, HPA）**
- **原則：** 根據觀察到的 CPU 利用率或其他自訂指標自動調整 Pod 副本數量。
- **Copilot 指引：** 為具有波動負載的無狀態應用程式建議使用 HPA。
- **設定：** `minReplicas`、`maxReplicas`、`targetCPUUtilizationPercentage`。

### **3. 垂直 Pod 自動調整器（Vertical Pod Autoscaler, VPA）**
- **原則：** 根據使用歷史自動調整容器的 CPU 和記憶體 requests/limits。
- **Copilot 指引：** 為了長期最佳化個別 Pod 的資源使用，建議使用 VPA。

## Kubernetes 的安全最佳實踐

### **1. 網路政策（Network Policies）**
- **原則：** 控制 Pod 和網路端點之間的通訊。
- **Copilot 指引：** 建議實作細粒度的網路政策（預設拒絕，例外允許），以限制 Pod 對 Pod 和 Pod 對外部通訊。

### **2. 角色型存取控制（Role-Based Access Control, RBAC）**
- **原則：** 控制誰可以在 Kubernetes 集群中做什麼。
- **Copilot 指引：** 定義細粒度的 `Role` 和 `ClusterRole`，然後使用 `RoleBinding` 和 `ClusterRoleBinding` 將它們繫結到 `ServiceAccount` 或使用者/群組。
- **最小權限：** 始終應用最小權限原則。

### **3. Pod 安全內容（Pod Security Context）**
- **原則：** 在 Pod 或容器層級定義安全設定。
- **Copilot 指引：**
    - 使用 `runAsNonRoot: true` 以防止容器以 root 身份執行。
    - 設定 `allowPrivilegeEscalation: false`。
    - 盡可能使用 `readOnlyRootFilesystem: true`。
    - 移除不需要的功能（`capabilities: drop: [ALL]`）。
- **範例（Pod 安全內容）：**
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
    - name: my-app
      image: my-repo/my-app:1.0.0
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
```

### **4. 映像安全（Image Security）**
- **原則：** 確保容器映像安全且無漏洞。
- **Copilot 指引：**
    - 使用可信任的最小基礎映像（distroless、alpine）。
    - 將映像漏洞掃描（Trivy、Clair、Snyk）整合到 CI 管道中。
    - 實作映像簽名和驗證。

### **5. API 伺服器安全（API Server Security）**
- **原則：** 保護對 Kubernetes API 伺服器的存取。
- **Copilot 指引：** 使用強式驗證（用戶端憑證、OIDC），強制執行 RBAC 並啟用 API 稽核。

## 日誌、監控和可觀測性

### **1. 集中式日誌（Centralized Logging）**
- **原則：** 從所有 Pod 收集日誌並集中進行分析。
- **Copilot 指引：**
    - 使用標準輸出（`STDOUT`/`STDERR`）記錄應用程式日誌。
    - 部署日誌代理（例如 Fluentd、Logstash、Loki），將日誌傳送到中央系統（ELK Stack、Splunk、Datadog）。

### **2. 指標收集（Metrics Collection）**
- **原則：** 從 Pod、節點和集群元件收集並儲存關鍵效能指標（KPI）。
- **Copilot 指引：**
    - 使用 Prometheus 搭配 `kube-state-metrics` 和 `node-exporter`。
    - 使用應用程式特定的出口工具定義自訂指標。
    - 設定 Grafana 進行視覺化。

### **3. 警報（Alerting）**
- **原則：** 為異常和重要事件設定警報。
- **Copilot 指引：**
    - 為基於規則的警報設定 Prometheus Alertmanager。
    - 為高錯誤率、資源可用性低、Pod 重新啟動和不健康探針設定警報。

### **4. 分散式追蹤（Distributed Tracing）**
- **原則：** 在集群內的多個微服務間追蹤請求。
- **Copilot 指引：** 實作 OpenTelemetry 或 Jaeger/Zipkin 進行端到端的請求追蹤。

## Kubernetes 中的部署策略

### **1. 滾動式更新（Rolling Updates，預設）**
- **原則：** 逐漸將舊版本的 Pod 替換為新版本。
- **Copilot 指引：** 這是 Deployment 的預設設定。設定 `maxSurge` 和 `maxUnavailable` 以進行精細控制。
- **優點：** 更新期間的停機時間最少。

### **2. 藍綠部署（Blue/Green Deployment）**
- **原則：** 執行兩個相同的環境（藍色和綠色）；完全切換流量。
- **Copilot 指引：** 為零停機發行建議使用。需要外部負載均衡器或 Ingress 控制器功能來管理流量切換。

### **3. 金絲雀部署（Canary Deployment）**
- **原則：** 在完整推展前，逐漸將新版本推出給少量使用者。
- **Copilot 指引：** 為使用真實流量測試新功能建議使用。使用 Service Mesh（Istio、Linkerd）或支援流量分割的 Ingress 控制器實作。

### **4. 回滾策略（Rollback Strategy）**
- **原則：** 能夠快速安全地回復到先前的穩定版本。
- **Copilot 指引：** 為 Deployment 使用 `kubectl rollout undo`。確保先前的映像版本可用。

## Kubernetes 宣告審查檢查清單

- [ ] `apiVersion` 和 `kind` 是否正確對應資源？
- [ ] `metadata.name` 是否具有描述性且遵循命名慣例？
- [ ] `labels` 和 `selectors` 是否一致使用？
- [ ] `replicas` 是否適當地設定工作負載？
- [ ] 是否為所有容器定義了 `resources`（requests/limits）？
- [ ] `livenessProbe` 和 `readinessProbe` 是否正確設定？
- [ ] 敏感配置是否透過 Secret 處理（而非 ConfigMap）？
- [ ] `readOnlyRootFilesystem: true` 是否盡可能設定？
- [ ] `runAsNonRoot: true` 和非 root 的 `runAsUser` 是否已定義？
- [ ] 不必要的 `capabilities` 是否已移除？
- [ ] 通訊限制是否考慮過 `NetworkPolicy`？
- [ ] RBAC 是否以最小權限為 ServiceAccount 設定？
- [ ] `ImagePullPolicy` 和映像標籤（避免 `:latest`）是否正確設定？
- [ ] 日誌是否發送到 `STDOUT`/`STDERR`？
- [ ] 排程時是否使用適當的 `nodeSelector` 或 `tolerations`？
- [ ] 滾動式更新的 `strategy` 是否已設定？
- [ ] 是否監控 `Deployment` 事件和 Pod 狀態？

## Kubernetes 常見問題故障排除

### **1. Pod 未啟動（Pending、CrashLoopBackOff）**
- 檢查 `kubectl describe pod <pod_name>` 以查看事件和錯誤訊息。
- 檢閱容器日誌（`kubectl logs <pod_name> -c <container_name>`）。
- 驗證資源 requests/limits 不會過低。
- 檢查映像拉取錯誤（映像名稱中有拼字錯誤、儲存庫存取）。
- 確保所需的 ConfigMap/Secret 已掛載且可存取。

### **2. Pod 未就緒（服務不可用）**
- 檢查 `readinessProbe` 配置。
- 驗證容器內的應用程式是否在預期的連接埠上接聽。
- 檢查 `kubectl describe service <service_name>` 以確保端點已連接。

### **3. Service 無法存取**
- 驗證 Service `selector` 是否符合 Pod 標籤。
- 檢查 Service `type`（內部使用 ClusterIP，外部使用 LoadBalancer）。
- 對於 Ingress，檢查 Ingress 控制器日誌和 Ingress 資源規則。
- 檢閱可能阻止流量的 `NetworkPolicy`。

### **4. 資源耗盡（OOMKilled）**
- 增加容器的 `memory.limits`。
- 最佳化應用程式的記憶體使用。
- 使用 `Vertical Pod Autoscaler` 建議最佳限制。

### **5. 效能問題**
- 使用 `kubectl top pod` 或 Prometheus 監控 CPU/記憶體使用。
- 檢查應用程式日誌中是否有查詢或操作緩慢的情況。
- 分析分散式追蹤以找出瓶頸。
- 檢查資料庫效能。

## 結論

在 Kubernetes 上部署應用程式需要深入了解其核心概念和最佳實踐。遵循這些有關 Pod、Deployment、Service、Ingress、配置、安全性和可觀測性的指引，你可以指導開發人員建構高度復原、可擴展和安全的雲原生應用程式。記住要持續監控、故障排除和改進 Kubernetes 部署，以獲得最佳效能和可靠性。

---

<!-- End of Kubernetes Deployment Best Practices Instructions -->

# Logging Architecture | 日誌架構設計

![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Overview ｜ 概述

This guide defines a scalable and reliable logging architecture for both native and containerized systems. It focuses on structured logging, centralized collection, and operational reliability.  
本指南說明如何設計一套可擴展且高可靠度的日誌架構，適用於原生系統與容器化環境。重點在於結構化日誌、集中化收集，以及營運穩定性。  

## Objectives of Logging Architecture | 日誌架構設計目標

A well-designed logging system enables troubleshooting, traceability, compliance, and system observability.  
良好的日誌系統應能支援問題排查、事件追蹤、法規遵循，以及整體系統觀測能力。  

**Key objectives / 主要目標**

* Fast troubleshooting (reduce MTTR) | 快速故障排除 (降低 MTTR)
* End-to-end traceability | 全流程追蹤
* Audit & compliance | 稽核與合規
* Monitoring and alerting integration | 整合監控告警
* Scalability | 可擴展性

## Core Architecture Overview | 核心架構概觀

All systems follow a standard logging pipeline from generation to analysis.  
所有系統的日誌處理從產生到分析皆遵循一致的流程。  

```
[Application / Service]
        ↓
[Collection / Forwarding Layer (with disk buffer)]
        ↓
[Message Queue (with ACKs)]
        ↓
[Centralized Storage]
        ↓
[Search: Query Engine]
        ↓
[Dashboard]
```

This model applies uniformly to both native and containerized environments.  
此模型同時適用於原生環境和容器化環境。

## Log Producer Layer | 日誌產生層  

All applications (native + containers) act as **log producers**, but they must remain **decoupled from storage**.   
所有應用程式（原生應用程式 + 容器應用程式）都充當**日誌生產者**，但它們必須與儲存**完全分離**。  
The key concept is:  
關鍵概念是：
> Applications generate logs, but never manage delivery or persistence.  
> 應用程式產生日誌，但無需管理日誌的傳遞或持久化。   

**Unified Output Strategy | 統一的輸出策略**

* **Native apps** → `journald`
* **Containers** → `stdout / stderr`

Example | 範例:

```bash
# Native
journalctl -u service-name

# Docker → always stdout
docker logs container_name
```


**Structured Logging | 結構化日誌**  

Logs must be structured, consistent, and traceable.  
日誌必須具備結構化、一致性與可追蹤性，以利後續分析與維運。  

Please refer to the [Audit Log and System Log](./AuditLog-SystemLog.md) for details.    
詳細內容請參考 [稽核日誌 與 系統日誌](./AuditLog-SystemLog.md)。  


### Container vs Native Logging Patterns | 容器日誌 與 原生系統日誌 模式比較

Different environments use different logging mechanisms, but share the same architectural goals.  
不同部署環境會採用不同的日誌方式，但最終目標（可觀測性與可靠性）是一致的。  

#### Native Pattern | 原生系統日誌

Applications write logs to files, then agents collect them.  
應用程式將日誌寫入本機檔案，再由代理程式收集。  

```
App → /var/log/app.log → Agent → Backend
```

**Advantages | 優點**

* Simple and predictable | 本地持久化（log 不易遺失）
* Mature tooling | 容易除錯（SSH + grep）

**Limitations | 缺點**

* Harder to scale | 擴展性較差
* Requires manual management | 需管理 log rotation

**File Layout | 檔案分布**  
```
/var/log/
├── system/
│   ├── syslog.log
│   ├── kernel.log
│   └── auth.log
│
├── application/
│   ├── robot-controller/
│   │   ├── app.log
│   │   ├── error.log
│   │   └── access.log
│   │
│   ├── efem-service/
│   │   ├── app.log
│   │   └── debug.log
│
├── middleware/
│   ├── kafka/
│   └── nginx/
│
└── custom/
    └── *.log
```

---

#### Container Pattern | 容器系統日誌

Logs are written to stdout/stderr and managed by container runtime.  
日誌透過 stdout/stderr 輸出，由容器平台負責管理與收集。  

```
Application → stdout/stderr → container runtime → Agent / Log Driver → Backend
```

Please refer to the [Agent-Based Collection vs Docker Log Driver](./AgentTailing-vs-LogDriver.md) for details.    
詳細內容請參考 [代理式收集 vs Docker 記錄驅動](./AgentTailing-vs-LogDriver.md) 。  

**Key Rule | 關鍵原則**  
- Applications MUST NOT write log files  
不可寫入容器內 log file  



**Advantages | 優點**

* Native support for distributed systems  
原生支援分散式系統  
* Easy aggregation  
易於統整  


**Limitations | 缺點**

* Logs tied to container lifecycle  
* Risk of loss without buffering  
容器重啟可能造成 log 遺失  
* Requires proper infrastructure setup  
需搭配收集機制  

**File Layout | 檔案分布**  
Option 1: Use Docker default log path  
選項 1：使用 Docker 預設日誌路徑  
```
/var/lib/docker/containers/
└── <container_id>/
    └── <container_id>-json.log
```
Option 2: Mounted volume    
選項 2：掛載磁碟區  
```
/shared/logs/
└── app/
    └── app.log
```
and container runs with  
並指定於容器啟動參數  
```
-v /shared/logs:/logs
```

#### Recommended Hybrid Model | 建議混合模式

Most real systems combine both approaches.  
實務系統通常會同時包含原生服務與容器化服務。  

```
File logs (Legacy services) + stdout logs (Container)
          ↓
      Agent / Log Driver
          ↓
      Central Storage
```

### Recommended Utilities | 推薦工具

| Category<br>類別        | Recommendation<br>建議              | Notes<br>備註                            |
| --------------- | --------------------------- | -------------------------------- |
| Logging Library<br>日誌庫 | `log4j2`, `logback` (Java)  | High performance, async support<br>高效能，支援非同步  |
|                 | `spdlog` (C++)              | Lightweight, fast<br>輕量級，快速                |
|                 | `winston`, `pino` (Node.js) | JSON-native logging<br>原生 JSON 日誌              |
| Log Format<br>日誌格式      | JSON                        | Mandatory for downstream parsing<br>下游解析的必要條件 |
| Context<br>上下文         | OpenTelemetry SDK           | Adds trace\_id correlation<br>新增 trace_id 關聯       |


## Log Collection Layer | 日誌收集層

A collection layer ensures reliable delivery of logs.  
日誌收集層負責將資料穩定傳送至後端系統。  



### Agent-Based Collection | 基於代理的收集

A lightweight **log agent (e.g., Fluent Bit)** acts as the **single collection and delivery pipeline**.   
This is the **most important component** in your design.   
一個輕量級的**日誌代理（例如 Fluent Bit）** 充當**單一的日誌收集和交付管道**。  
這是您設計中**最重要的元件**。

> The agent absorbs complexity so applications stay simple and robust.  
> 代理程式會吸收複雜性，從而使應用程式保持簡潔和健壯。


**Flow | 流程**

```
Logs → Agent → Buffer → Retry → Backend
```

### Design Requirements | 設計要求

* Local buffering (for network failure)  
本地緩衝（應對網路故障）  
* Retry with backoff  
退避重試機制  
* Decoupling from application  
與應用程式解耦


Please refer to the [Zero‑Log‑Loss Pipeline](./Zero‑Log‑Loss-Pipeline.md) for details.    
詳細內容請參考 [日誌零遺失管線](./Zero‑Log‑Loss-Pipeline.md) 。  

### Recommended Utilities | 推薦工具

| Tool<br>工具                           | Strength<br>優點                        | Notes<br>備註                         |
| ------------------------------ | ------------------------------- | ----------------------------- |
| **Fluent Bit** (Recommended)（建議） | Lightweight, ideal for embedded<br>輕量級，嵌入式應用的理想選擇 | Cortex-A friendly<br>支援 Cortex-A             |
| Fluentd                        | More powerful but heavier<br>功能更強大，但體積更大       | Use if complex routing needed<br>適用於需要複雜佈線的情況 |
| Filebeat                       | Simple, stable<br>簡單穩定                  | Good ELK integration<br>ELK 整合良好          |
| Vector (Datadog OSS)<br>（Datadog 開源軟體）           | Modern, fast<br>現代、快速                    | Strong performance<br>效能強勁            |


**Fluent Bit Collection Example | 收集設定範例**  
```
# Native System
[INPUT]
    Name tail
    Path /var/log/application/*/*.log
    Tag app.*

[INPUT]
    Name systemd
    Tag system.*

# Container
[INPUT]
    Name      tail
    Path      /var/lib/docker/containers/*/*.log
    Parser    docker
```

### Why not direct DB logging? | 為什麼不直接記錄到資料庫？  

**Feedback amplification failure** (very common failure mode)  
**回應放大故障**（非常常見的故障模式）  

```
Exception occurs → app logs heavily → DB overloaded →   
發生異常 → 應用程式日誌量巨大 → 資料庫過載 →  

logging becomes slower → app becomes slower →
日誌記錄速度變慢 → 應用運轉速度變慢 →

more timeouts → more logs → meltdown loop  
逾時次數增多 → 記錄增多 → 崩潰循環
```

* 請參考說明影片：[碧海潮生曲的系統架構學 -- 從《射鵰英雄傳》看《易經》、五音、五行到狀態機](https://youtu.be/4RROHItZuWc)


| Issue<br>問題          | Without agent<br>不使用代理    | With agent<br>使用代理    |
| -------------- | --------------------- | --------------------- |
| DB slow<br>資料庫運行緩慢        | application slows<br>應用程式運行緩慢     |  buffered<br>緩衝 |
| DB down<br>資料庫無運作        | logs lost / app error<br>日誌遺失/應用程式錯誤 | retry<br>重試    |
| high log burst<br>日誌爆發 | system instability<br>系統不穩定    | smoothed<br>平滑 |


Please refer to the [Logging vs Relational DB in a Logging Pipeline](./Comparison-in-LoggingPipeline.md) for details.    
詳細內容請參考 [日誌管線中的 Logging vs 關聯式資料庫](./Comparison-in-LoggingPipeline.md) 。  

## Message Queue Phase (Decoupling Layer) | 訊息佇列階段（解耦層）

Provide **asynchronous, scalable, fault-tolerant transport**  
提供**非同步、可擴充、容錯的傳輸**  

### Key Concepts | 關鍵概念

* Prevent backpressure from storage  
防止儲存反壓  
* Enable **multi-consumer pipelines**  
支援**多消費者管道**  
* Durable buffering  
持久緩衝  

Please refer to the [Zero‑Log‑Loss Pipeline](./Zero‑Log‑Loss-Pipeline.md) for details.    
詳細內容請參考 [日誌零遺失管線](./Zero‑Log‑Loss-Pipeline.md) 。  

### Recommended Utilities | 推薦工具

| Tool<br>工具           | Best Use Case<br>最佳實例                     | Notes<br>備註                    |
| -------------- | --------------------------------- | ------------------------ |
| **Kafka**     | High throughput, industrial scale<br>高吞吐量，工業級規模 | Industry standard<br>業界標準        |
| RabbitMQ       | Lower throughput, simpler routing<br>吞吐量較低，路由更簡單 | Easier to manage<br>更易於管理         |
| NATS JetStream | Lightweight, low latency<br>輕量級，低延遲          | Good for edge systems<br>適用於邊緣系統    |
| Redis Streams  | Simple pipeline<br>簡單的管線                   | Not for large scale logs<br>不適用於大規模日誌 |


### Design Notes | 設計說明

* Use **topic per log type** (`system`, `app`, `security`)  
**一個日誌類型用一個主題**（`system`、`app`、`security`）  
* Enable **retention policy** (e.g., 24–72 hrs)  
啟用**保留策略**（例如，24-72 小時）  


## Centralized Storage (Persistence Layer) | 集中式儲存

Logs should be centrally stored for querying and analysis.  
日誌應集中儲存，以利查詢與分析。  

### Key Concepts | 關鍵概念

* Separation between **hot storage** and **archival**  
**熱儲存** 與 **歸檔儲存** 分離  
* Indexing strategy is critical  
索引策略至關重要  
* Avoid DB for raw logs (use search engines instead)  
避免將原始日誌儲存在資料庫中（應使用搜尋引擎）  


### Recommended Utilities | 推薦工具

| Tool<br>工具                             | Strength<br>優點                    | Notes<br>備註                       |
| -------------------------------- | --------------------------- | --------------------------- |
| **Elasticsearch / OpenSearch** ✅ | Full-text search, indexing<br>全文搜尋、索引  | Core logging DB<br>核心日誌資料庫             |
| Loki (Grafana)                   | Cost-effective, label-based<br>經濟高效，基於標籤 | Good alternative<br>不錯的替代方案            |
| ClickHouse                       | High compression, analytics<br>高壓縮率、分析 | Good for metrics/log hybrid<br>適用於指標/日誌混合儲存 |
| Object Storage (S3/MinIO)<br>物件儲存 (S3/MinIO)        | Archive<br>歸檔                     | Cheap long-term storage<br>低成本的長期儲存     |


### Storage Design | 儲存設計

**Retention Policy | 保留策略**

| Log Type<br>日誌類型    | Retention<br>保留期限 |
| ----------- | --------- |
| Debug<br>除錯       | 1–3 days  |
| Application<br>應用 | 7–30 days |
| Audit<br>審查       | 90+ days  |


**Index Strategy (Example) | 索引策略（範例）** 

```
logs-{service}-{date}
```

**Tiered Storage | 分層儲存**

* Hot → recent logs  
熱儲存 → 最新日誌
* Warm → older logs  
溫儲存 → 較舊日誌
* Cold → archive  
冷儲儲 → 歸檔日誌  

## Search Phase (Query & Analysis) | 搜尋階段（查詢與分析）  

Provide fast log querying, filtering, correlation  
提供快速的日誌查詢、過濾和關聯功能  

### Key Concepts | 關鍵概念  

* Full-text search  
全文搜尋  
* Time-series filtering  
時間序列過濾  
* Correlation via `trace_id`  
透過 `trace_id` 進行關聯  


### Recommended Utilities | 推薦工具

| Tool<br>工具                          | Notes<br>註釋                            |
| ----------------------------- | -------------------------------- |
| **Elasticsearch Query DSL**<br>**Elasticsearch 查詢 DSL**   | Powerful but complex<br>功能強大但較為複雜              |
| OpenSearch Dashboards<br>OpenSearch 儀表板         | Integrated experience<br>整合體驗            |
| Grafana (Loki/Elastic plugin)<br>Grafana（Loki/Elastic 外掛程式） | Unified observability<br>統一可觀測性            |
| Kibana                        | Classic ELK visualization/search<br>經典的 ELK 視覺化/搜尋 |


### Typical Queries | 典型查詢

* Error rate over time  
錯誤率隨時間的變化  
* Service-specific logs  
特定服務的日誌  
* Trace-based debugging  
基於追蹤的調試  


## Dashboard Phase (Visualization & Monitoring) | 儀表板階段（視覺化與監控）  

Convert logs into **actionable insights**  
將日誌轉化為**可執行的洞察**  

### Key Concepts | 關鍵概念  

* Real-time observability  
即時可觀測性  
* Alerting & anomaly detection  
警報與異常偵測  
* Role-based dashboards (Ops / Dev / Manager)  
角色為基礎的儀錶板（維運/開發/管理）  


### Recommended Utilities | 推薦工具

| Tool<br>工具                      | Strength<br>優勢                    |
| ------------------------- | --------------------------- |
| **Grafana** ✅             | Best for unified dashboards<br>最適合統一儀錶板 |
| Kibana                    | Built-in with Elastic<br>內建 Elastic       |
| OpenSearch Dashboards     | Open-source Kibana fork<br>Kibana 的開源分支     |
| Prometheus + Alertmanager | Metrics + alerting<br>指標 + 警報          |

### Dashboard Examples | 儀表板範例

* System health overview  
系統健康狀況概覽  
* Error heatmap  
錯誤熱圖  
* Throughput per service  
各服務吞吐量  
* Container failure tracking  
容器故障追蹤  


## General Design Consideration | 通用設計考量

### Reliability Considerations | 可靠度設計

Log systems must handle failures and network issues.  
日誌系統需能因應網路與系統異常。  

**Key Mechanisms | 關鍵機制**

* Buffer | 緩衝機制
* Retry | 重試
* Async | 非同步



### Security and Compliance | 安全與合規

Sensitive data must not be logged.  
不可記錄敏感資訊。  
- Passwords / Tokens / Personal data  
密碼 / Token / 個資  

**Controls | 控制措施**

* Mask sensitive fields  
屏蔽敏感字段  
* Enforce role-based access  
強制執行基於角色的存取控制  
* Audit log access itself  
稽核日誌存取記錄  

### Performance Considerations | 效能考量

* Non-blocking logging  
非阻塞    
* Avoid excessive DEBUG logs  
控制 DEBUG log  
* Batch transmissions  
批次傳輸  



### Common Anti-Patterns | 常見錯誤

**General | 通用**

* Plain text logs  
純文字日誌  
* No correlation IDs  
無關聯 ID  
* No retention policy  
無保留策略


**Container | 容器系統**

* Writing logs to files inside containers  
將日誌寫入容器內部的文件  
* Ignoring stdout collection  
忽略標準輸出收集  
* No monitoring of log growth  
不監控日誌成長  

**Native | 原生系統**

* No log rotation  
無日誌輪換  
* Logs scattered across directories  
日誌分散在各目錄中  
* No central aggregation  
無集中統整


---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Logging Architecture © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌架構設計 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

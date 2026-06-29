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
App → /var/log/app.log → Agent / Log Driver → Backend
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
一個輕量級的**日誌代理（例如 Fluent Bit）**充當**單一的日誌收集和交付管道**。  
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
詳細內容請參考 [零日誌遺失管線](./Zero‑Log‑Loss-Pipeline.md) 。  

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

Please refer to the [Docker Log Driver vs Fluent Bit Tailing](./LogDriver-vs-AgentTailing.md) for details.    
詳細內容請參考 [Docker 記錄驅動 vs Fluent Bit 日誌擷取](./LogDriver-vs-AgentTailing.md) 。  

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
詳細內容請參考 [零日誌遺失管線](./Zero‑Log‑Loss-Pipeline.md) 。  

### Recommended Utilities | 推薦工具

| Tool           | Best Use Case                     | Notes                    |
| -------------- | --------------------------------- | ------------------------ |
| **Kafka**     | High throughput, industrial scale | Industry standard        |
| RabbitMQ       | Lower throughput, simpler routing | Easier to manage         |
| NATS JetStream | Lightweight, low latency          | Good for edge systems    |
| Redis Streams  | Simple pipeline                   | Not for large scale logs |

### Design Notes

* Use **topic per log type** (`system`, `app`, `security`)
* Enable **retention policy** (e.g., 24–72 hrs)



## Centralized Storage (Persistence Layer) | 集中式儲存

Logs should be centrally stored for querying and analysis.  
日誌應集中儲存，以利查詢與分析。  

### Key Concepts

* Separation between **hot storage** and **archival**
* Indexing strategy is critical
* Avoid DB for raw logs (use search engines instead)

### Recommended Utilities | 推薦工具

| Tool                             | Strength                    | Notes                       |
| -------------------------------- | --------------------------- | --------------------------- |
| **Elasticsearch / OpenSearch** ✅ | Full-text search, indexing  | Core logging DB             |
| Loki (Grafana)                   | Cost-effective, label-based | Good alternative            |
| ClickHouse                       | High compression, analytics | Good for metrics/log hybrid |
| Object Storage (S3/MinIO)        | Archive                     | Cheap long-term storage     |


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

## Search Phase (Query & Analysis)

Provide fast log querying, filtering, correlation

### Key Concepts

* Full-text search
* Time-series filtering
* Correlation via `trace_id`

### Recommended Utilities | 推薦工具

| Tool                          | Notes                            |
| ----------------------------- | -------------------------------- |
| **Elasticsearch Query DSL**   | Powerful but complex             |
| OpenSearch Dashboards         | Integrated experience            |
| Grafana (Loki/Elastic plugin) | Unified observability            |
| Kibana                        | Classic ELK visualization/search |

### Typical Queries

* Error rate over time
* Service-specific logs
* Trace-based debugging

## Dashboard Phase (Visualization & Monitoring)

Convert logs into **actionable insights**

### Key Concepts

* Real-time observability
* Alerting & anomaly detection
* Role-based dashboards (Ops / Dev / Manager)

### Recommended Utilities | 推薦工具

| Tool                      | Strength                    |
| ------------------------- | --------------------------- |
| **Grafana** ✅             | Best for unified dashboards |
| Kibana                    | Built-in with Elastic       |
| OpenSearch Dashboards     | Open-source Kibana fork     |
| Prometheus + Alertmanager | Metrics + alerting          |

### Dashboard Examples

* System health overview
* Error heatmap
* Throughput per service
* Container failure tracking

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

![BY NC ND](../../../img/Cc-by-nc-sa.png)  
Logging Architecture © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌架構設計 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

# Logging Architecture | 日誌架構設計

![Practical System Design Considerations](../img/PracticalSystemDesignConsiderations.png)  

## Overview ｜ 概述

This guide defines a scalable and reliable logging architecture for both normal and containerized systems. It focuses on structured logging, centralized collection, and operational reliability.  
本指南說明如何設計一套可擴展且高可靠度的日誌架構，適用於一般系統與容器化環境。重點在於結構化日誌、集中化收集，以及營運穩定性。  

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
[Structured Log Generation]
        ↓
[Collection / Forwarding Layer]
        ↓
[Centralized Storage]
        ↓
[Analysis / Visualization / Alerting]
```

This model applies uniformly to both normal and containerized environments.  
此模型同樣適用於一般環境和容器化環境。
  
### Best Practice | 最佳實務

**Container | 容器**

* stdout
* DaemonSet/Agent

**Normal | 一般**

* `/var/log`
* log rotation

## Log Generation (Application Layer) | 日誌產生（應用層）

Logs must be structured, consistent, and traceable.  
日誌必須具備結構化、一致性與可追蹤性，以利後續分析與維運。  

**Structured Logging | 結構化日誌**

Please refer to the [Audit Log and System Log](./AuditLog-SystemLog.md) for details.    
詳細內容請參考 [稽核日誌 與 系統日誌](./AuditLog-SystemLog.md)。  


## Container vs Normal Logging Patterns | 容器 與 一般日誌模式比較

Different environments use different logging mechanisms, but share the same architectural goals.  
不同部署環境會採用不同的日誌方式，但最終目標（可觀測性與可靠性）是一致的。  

### Normal Pattern | 一般系統

Applications write logs to files, then agents collect them.  
應用程式將日誌寫入本機檔案，再由代理程式收集。  

```
App → /var/log/app.log → Fluent Bit → Backend
```

**Advantages | 優點**

* Simple and predictable | 本地持久化（log 不易遺失）
* Mature tooling | 容易除錯（SSH + grep）

**Limitations | 缺點**

* Harder to scale | 擴展性較差
* Requires manual management | 需管理 log rotation



### Container Pattern | 容器系統

Logs are written to stdout/stderr and managed by container runtime.  
日誌透過 stdout/stderr 輸出，由容器平台負責管理與收集。  

```
Container → stdout → Agent / Log Driver → Backend
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



### Key Differences | 差異比較

| Aspect 項目   | Normal 一般 | Container 容器 |
| ---- | -------- | ------ |
| Log storage <br>儲存方式 | File <br>檔案       | stdout stream |
| Persistence <br>持久性  | High <br>高        | Depends on collection <br>取決於收集  |
| Debugging<br>除錯機制   | Local access easy<br>本地存取 | Needs tooling<br>支援工具         |
| Scalability<br>擴展性  | Limited<br>有限       | Strong<br>高      |
| Metadata<br>管理方式 | Manual<br>手動       | Built-in labels <br>平台化    |



### Recommended Hybrid Model | 建議混合模式

Most real systems combine both approaches.  
實務系統通常會同時包含一般服務與容器化服務。  

```
File logs (Legacy services) + stdout logs (Container)
          ↓
      Fluent Bit
          ↓
   Central Storage
```



## Log Collection Layer | 日誌收集層

A collection layer ensures reliable delivery of logs.  
日誌收集層負責將資料穩定傳送至後端系統。  

### Recommended Approach: Agent-Based Collection | 推薦方法：基於代理的收集

**Tools | 建議工具**

* Fluent Bit (lightweight, edge-friendly) | （輕量、適合 edge）
* Fluentd
* Filebeat

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

## Centralized Log Storage | 集中式日誌儲存

Logs should be centrally stored for querying and analysis.  
日誌應集中儲存，以利查詢與分析。  

**Common Platforms | 常見方案**

* ELK / EFK
* Loki + Grafana
* Cloud logging

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

## Log Processing and Enrichment | 日誌處理與強化

Logs are enriched with metadata for better analysis.  
透過補充額外欄位，提升日誌分析能力。  

Examples | 例如

* Environment (prod/staging)
* Region
* Machine ID
* Application version

Please refer to the **Log Format** of [Audit Log and System Log](./AuditLog-SystemLog.md) for details.    
詳細內容請參考 [稽核日誌 與 系統日誌](./AuditLog-SystemLog.md) 的［日誌格式］一節。  

## Query, Visualization, and Alerting | 查詢、視覺化與告警

Logs are used for search, dashboards, and alerts.  
透過日誌進行查詢、監控儀表板與告警設定。  

**Tools | 工具**

* Grafana / Kibana
* Azure Monitor / Splunk

**Use Cases | 使用範例**

**Search | 搜尋**

```
service="robot-controller" AND level="ERROR"
```

**Dashboards | 儀錶板**

* Error rate
* Failure trends
* Latency

**Alerting | 告警**

```
ERROR rate > threshold → trigger alert
```


## Reliability Considerations | 可靠度設計

Log systems must handle failures and network issues.  
日誌系統需能因應網路與系統異常。  

**Key Mechanisms | 關鍵機制**

* Buffer | 緩衝機制
* Retry | 重試
* Async | 非同步



## Security and Compliance | 安全與合規

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

## Performance Considerations | 效能考量

* Non-blocking logging  
非阻塞    
* Avoid excessive DEBUG logs  
控制 DEBUG log  
* Batch transmissions  
批次傳輸  



## Common Anti-Patterns | 常見錯誤

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

**Normal | 一般系統**

* No log rotation  
無日誌輪換  
* Logs scattered across directories  
日誌分散在各目錄中  
* No central aggregation  
無集中統整


---

# License｜授權條款

![BY NC ND](../../img/Cc-by-nc-sa.png)  
Logging Architecture © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌架構設計 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

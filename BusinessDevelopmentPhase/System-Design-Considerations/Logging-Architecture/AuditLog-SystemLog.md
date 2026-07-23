# Audit Log and System Log | 稽核日誌 與 系統日誌

![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Overview ｜ 概述  

**Audit logs** and **system logs** should serve different purposes: audit logs track user/business activities, while system logs focus on system health and critical errors. This separation prevents interference between high-volume logs and critical operations.  
**稽核日誌** 與 **系統日誌** 應分工明確：前者記錄使用者與商務行為，後者著重在系統運作與關鍵錯誤。此設計可避免大量日誌影響系統關鍵功能。  

## Audit Log | 稽核日誌

### Audit Log Overview | 稽核日誌概述
Audit logs are not just for developer debugging—they primarily ensure traceability, accountability, and help users troubleshoot issues efficiently. Clear and user-friendly logs improve both user experience and system maintainability.  
稽核日誌不僅用於開發者除錯，其核心目的在於確保可追溯性與責任歸屬，並協助使用者有效排除問題。清晰且易於理解的日誌能提升使用者體驗與系統可維護性。  

### Event Content | 日誌內容
Each log entry must clearly describe who did what, when, where, and the result. A standardized structure (e.g., actor, action, target, status, timestamp, source, correlation ID) ensures effective traceability and root cause analysis.  
每筆日誌需清楚記錄「誰、在何時何地、執行了什麼操作，以及結果為何」。透過標準化欄位（如使用者、操作、目標、狀態、時間、來源與關聯識別碼），可有效支援追蹤與根因分析。   

Logs should define severity levels (INFO, WARNING, ERROR, CRITICAL) and use clear, human-readable language. Avoid excessive technical jargon and provide actionable guidance when possible.  
日誌應定義嚴重等級（資訊、警告、錯誤、嚴重），並使用易於理解的文字，避免過多技術術語，並在可能時提供可行的解決建議。   

#### Timestamp | 時間戳記
Accurate timestamps are essential and should be synchronized using trusted sources such as NTP. For offline systems, solutions like RTC, GPS, local NTP servers, or hybrid logging help ensure long-term reliability of time data.  
精確的時間戳記至關重要，應透過 NTP 等可信來源進行同步。對於離線系統，可採用 RTC、GPS、本地 NTP 伺服器或混合式記錄方式，以確保時間的長期準確性。  

##### Timestamp Policy | 時間戳記政策
System time must not be modified to maintain consistency and trustworthiness of logs. If needed, a separate custom timestamp should be introduced, with recorded time differences to preserve traceability and integrity.  
系統時間不可被修改，以維持日誌的一致性與可信度。如有需求，應使用獨立的自訂時間欄位，並記錄時間差，以確保可追溯性與完整性。  

#### Log Format | 日誌格式

A structured log format ensures consistency, traceability, and integration with external systems.   
結構化日誌格式可確保一致性與可追溯性，並便於系統整合。  

**Required Fields | 必要欄位**

* Timestamp | 時間戳記
* Service name | 服務名稱
* Trace ID | 追蹤編號
* Log level | 等級
* Message | 訊息內容

**Standard Fields | 標準欄位**

```
{
  "timestamp": "ISO 8601 format",
  "level": "INFO | WARNING | ERROR | CRITICAL",
  "service": "Service or module name",
  "host": "Hostname or device ID",
  "source": "IP or origin",
  "actor": "User ID or system actor",
  "action": "Operation performed",
  "target": "Resource or object",
  "result": "SUCCESS | FAILURE",
  "error_code": "Optional error code",
  "message": "Human-readable description",
  "correlation_id": "Trace ID for linking events",
  "custom_timestamp": "Optional customer-defined time",
  "time_delta_ms": "Difference between system and custom time",
  "metadata": {
    "key": "Additional structured data"
  }
}
```

**Field Description | 欄位說明**

- **timestamp**：System-generated authoritative time（系統權威時間）  
- **level**：[Severity level（嚴重等級）](../Priority.md)  
- **service**：Originating module/service（來源模組）  
- **actor**：User or system identity（操作主體）  
- **action**：What happened（操作行為）  
- **target**：Affected resource（影響對象）  
- **result**：Outcome status（執行結果）  
- **correlation_id**：Trace across services（跨服務追蹤）  
- **metadata**：Extended structured details（擴充資料）  

##### Audit Log Examples | 稽核日誌範例

ogin Failure | 登入失敗
```
{
  "timestamp": "2026-06-16T14:02:10Z",
  "level": "WARNING",
  "service": "audit-service",
  "actor": "unknown",
  "action": "LOGIN",
  "target": "auth-system",
  "result": "FAILURE",
  "error_code": "AUTH_401",
  "source": "203.0.113.5",
  "correlation_id": "login-456",
  "message": "Invalid username or password"
}
```

### Log Integrity | 日誌完整性
Audit logs must be protected from tampering using mechanisms such as append-only storage, hash chaining, digital signatures, and WORM storage to ensure reliability for audits and forensics.  
稽核日誌必須防止竄改，可透過僅新增寫入、雜湊鏈結、數位簽章與不可修改儲存等機制，確保其在稽核與鑑識分析中的可信度。  

Access to audit logs must be strictly restricted. Only authorized roles can view or export logs, modifications are prohibited, and all access activities must be recorded.  
日誌存取必須嚴格控管，僅授權角色可檢視或匯出，禁止修改，且所有存取行為皆需被記錄。  

Logs should have a clearly defined lifecycle, including retention duration, rotation, archiving, and traceable deletion mechanisms.  
日誌需具備明確的生命週期管理，包括保存期限、輪替機制、歸檔策略，以及可追蹤的刪除流程。

Logs should support export mechanisms (e.g., files, APIs, Syslog) and integration with external systems, including centralized logging and SIEM solutions if needed.  
日誌應支援多種匯出方式（如檔案、API、Syslog），並可與外部系統整合，例如集中式日誌系統或資安事件管理（SIEM）。  




## System Log | 系統日誌

### System Log Overview | 系統日誌概述
System logs focus on system behavior, debugging, and operational monitoring.  
系統日誌著重系統內部行為、除錯與運維監控。  

### Event Content | 日誌內容
System logs capture technical runtime information for troubleshooting and performance analysis.  
系統日誌記錄技術性執行資訊，用於除錯與效能分析。  

#### Log Format | 日誌格式

A structured log format ensures consistency, traceability, and integration with external systems.   
結構化日誌格式可確保一致性與可追溯性，並便於系統整合。  

**Standard Fields | 標準欄位**
```
{
  "timestamp": "2026-06-16T14:20:00Z",
  "level": "TRACE | DEBUG | INFO | WARNING | ERROR | CRITICAL",
  "service": "module/service name",
  "host": "hostname or device id",
  "process_id": 1234,
  "thread_id": 56,
  "module": "component name",
  "event": "event type",
  "message": "log message",
  "error_code": "optional",
  "stack_trace": "optional",
  "metrics": {
    "cpu": "optional",
    "memory": "optional",
    "latency_ms": "optional"
  },
  "correlation_id": "trace id"
}
```


##### System Log Examples | 系統日誌範例

Service Startup | 服務啟動   
```
{
  "timestamp": "2026-06-16T14:00:00Z",
  "level": "INFO",
  "service": "order-service",
  "host": "machine-01",
  "process_id": 1024,
  "thread_id": 1,
  "module": "startup",
  "event": "SERVICE_START",
  "message": "Order service started successfully",
  "correlation_id": "sys-001"
}
```

### Log Retention Policy | 保存政策
System logs are high-volume and typically use short-term rotation.  
系統日誌量大，通常採短期保存與輪替策略。  

### Severity & Debug Levels | 嚴重性與除錯等級
System logs include detailed levels such as DEBUG and TRACE for troubleshooting.  
系統日誌包含 DEBUG、TRACE 等細粒度等級以支援除錯。  
Please refer to the [Priority](../Priority.md) for details.  
詳細內容請參考 [重要性權重](../Priority.md)。   

## Comparison Table | 比較表

| Aspect | Audit Log | System Log | 說明 |
|--------|----------|-----------|------|
| **Primary Purpose** | Traceability, compliance | Debugging, monitoring | 稽核 vs 運維 |
| **Focus** | Who did what | System behavior | 人 vs 系統 |
| **Content** | actor, action, result | process, thread, metrics | 商務 vs 技術 |
| **Structure** | Strict | Flexible | 統一 vs 彈性 |
| **Integrity** | Required | Optional | 必須 vs 視需求 |
| **Readability** | User-friendly | Technical | 易讀 vs 技術 |
| **Volume** | Moderate | High | 中 vs 高 |
| **Retention** | Long-term | Short-term | 長 vs 短 |
| **Severity Levels** | INFO / WARNING / ERROR | DEBUG / TRACE / INFO | 粗 vs 細 |
| **Access Control** | Strict | Less strict | 嚴格 vs 寬鬆 |
| **Modification** | Not allowed | Allowed (rotation) | 不可 vs 可清理 |
| **Architecture** | Dedicated service | Logging pipeline | 專用服務 vs 管線 |

## Load Separation Principle | 負載分流原則  

**Audit logs** and **system logs** must be handled by separate services to avoid performance bottlenecks and improve scalability. Decoupling logging from core operations ensures stability under heavy load.  
**稽核日誌** 與 **系統日誌** 應由不同服務處理，以避免資源競爭，並提升可擴展性。將日誌與核心系統解耦可在高負載下維持穩定性。  

Logging should use asynchronous processing, buffering, and rate control to prevent overload. Mechanisms like queues, backpressure, and sampling ensure logs do not block or degrade system performance.  
日誌應採用非同步處理、緩衝與流量控制，以避免過載。透過佇列、回壓與取樣等機制，確保日誌不會阻塞或影響系統效能。  

### Architecture Patterns (Reference: Linux) | 架構模式（參考 Linux）
Similar to Linux logging (e.g., syslog/journald), logs are collected through a unified interface and routed to different backends. Logging services operate independently, and critical logs are prioritized.   
類似 Linux（如 syslog/journald）的設計，日誌透過統一介面收集並分流至不同後端，且日誌服務獨立運作，並優先處理關鍵訊息。  



## Log & System Mode Mapping | 日誌與系統模式對應表


| [System Mode<br>系統模式](../System-Mode-and-State.md)          | Goal / Intent<br>目標 / 意圖                                               | System Log Behavior<br>系統日誌行為                                                                       | Audit Log Behavior<br>稽核日誌行為                                                                           | Logging Level<br>日誌等級     | Rationale / Why<br>原因說明                                                                       |
| ------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------------- | ------------------------------------------------------------------------------------------ |
| **Running Mode<br>運行模式**     | Performance-first<br>效能優先<br>Stability, performance, high availability<br>穩定性、效能、高可用性            | - Log only critical errors<br>- Basic health monitoring<br>- Avoid high-frequency/internal logs<br>僅記錄關鍵錯誤、基本健康監控，避免高頻或內部細節記錄               | - Record user actions<br>- Track business operations<br>- Maintain accountability<br>記錄使用者行為與業務操作，確保可追溯性          | ERROR / WARN           | Minimize performance impact and logging overhead<br>降低記錄對效能的影響，符合正式運行效率需求                  |
| **Manual Mode<br>手動模式**      | Visibility-first<br>可視性優先<br>Debugging, diagnostics, troubleshooting<br>除錯、診斷、故障排除               | - Full internal logs (state, flow, function calls)<br>- Enable diagnostics<br>- Support step execution info<br>完整內部行為記錄（狀態/流程/函式），支援診斷與逐步執行 | - Detailed operator actions<br>- Trace troubleshooting steps<br>- Capture debugging scenarios<br>詳細記錄操作員行為與故障排除過程 | DEBUG（開發）<br>TRACE（現場） | Maximize visibility for root cause analysis; performance is secondary<br>提升可視性以利根因分析，效能非優先 |
| **Maintenance Mode<br>維護模式** | Traceability-first<br>可追溯性優先<br>System updates, configuration, controlled changes<br>系統更新、設定變更、受控操作 | - Log upgrade steps, migration status<br>- Record system transitions<br>- Capture service start/stop<br>記錄升級流程、資料移轉、服務啟停                    | - Track admin operations<br>- Record config changes & patches<br>- Maintain audit trail<br>記錄管理員操作、設定變更與修補歷程      | INFO / WARN / TRACE    | Ensure traceability, compliance, and rollback capability<br>確保可追溯性、合規性與回復能力                |
| **Simulation Mode<br>模擬模式**  | Flexibility-first<br>彈性優先<br>Testing, validation, training without hardware<br>無實體硬體的測試、驗證與訓練    | - Log simulated events<br>- Capture virtual hardware interactions<br>- Record test scenarios<br>記錄模擬事件與虛擬設備互動                               | - Optional logging<br>- Record training/test user actions<br>視需求記錄訓練或測試行為                                         | DEBUG / INFO           | High verbosity supports validation and development<br>高詳細度有助於測試與開發驗證                       |
| **Safe Mode<br>安全模式**        | Safety-first<br>安全優先<br>Safety protection under faults or hazards<br>發生故障時的安全保護             | - Log fault triggers<br>- Record safety actions (shutdown, stop)<br>- Capture recovery attempts<br>記錄故障觸發、安全動作與復原過程                         | - Minimal but critical logs<br>- Record acknowledgements or overrides<br>僅記錄重要操作（如確認或人工介入）                        | ERROR / CRITICAL       | Focus on safety and fault tracking; avoid overload during failure<br>以安全與故障追蹤為核心，避免系統過載    |

### Practical Implementation Tip | 實用技巧
You can implement this with a **mode-aware logging configuration**:  
您可以使用**模式感知日誌設定**來實現此功能：  

```
if (mode == RUNNING):
    system_log.level = ERROR
elif (mode == MANUAL):
    system_log.level = DEBUG
elif (mode == SAFE):
    system_log.level = CRITICAL
...
```

Or use:  
或使用：  

- Runtime config switch   
運轉時設定開關  
- Feature flags  
功能標誌  
- Central logging controller  
中央日誌控制器    

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Logging Architecture © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌架構設計 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際版](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

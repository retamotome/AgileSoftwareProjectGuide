# Logging vs Relational DB in a Logging Pipeline | 日誌管線中的 Logging vs 關聯式資料庫

![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Purpose and Design Philosophy | 設計目的與核心理念

A **logging system** is designed for high-throughput, append-only data ingestion where events are written continuously and rarely modified. The primary goal is to capture system behavior, operational signals, and diagnostic information with minimal latency or data loss. Logs are typically treated as time-series or event streams, optimized for sequential writes and efficient aggregation during search and analysis.

**日誌系統（Logging System）** 的設計目的是處理高吞吐量、僅追加（append-only）的資料寫入，事件會持續產生且幾乎不會被修改。其主要目標是在低延遲與低資料遺失風險下，完整記錄系統行為、營運訊號及除錯資訊。日誌通常被視為時間序列或事件流，並針對順序寫入與後續搜尋分析的聚合效率進行最佳化。

In contrast, a **relational database (RDB)** is designed for structured data with strong consistency, transactional guarantees, and normalized relationships. Its purpose is to support business logic, enforce schema constraints, and enable precise querying and updates.

相較之下，**關聯式資料庫（RDB）** 則是為結構化資料而設計，強調一致性（Consistency）、交易保證（Transaction）以及正規化關聯。其核心用途在於支撐商業邏輯、強制資料結構規範（Schema），並提供精確查詢與資料更新能力。

This difference explains why logging systems emphasize **write scalability and analysis**, while relational databases emphasize **data correctness and structured querying**.

這也說明了為什麼日誌系統重視「高寫入擴展性與後續分析」，而關聯式資料庫則重視「資料正確性與結構化查詢」。

## Behavior in a Logging Pipeline | 在日誌管線中的行為差異

In a typical logging pipeline (Application → Collection → Queue → Storage → Search → Dashboard), storage behaves very differently depending on the system used.

在標準的日誌架構（應用程式 → 收集 → 佇列 → 儲存 → 搜尋 → 儀表板）中，儲存層的運作方式會因為選用系統不同而產生顯著差異。

With a **logging system**, logs flow through collectors and queues into a storage layer optimized for continuous ingestion. Data is written in streaming or batch form, indexed by time or metadata, and designed to handle large volumes efficiently.

使用**日誌系統**時，資料會透過收集器與佇列流入一個專為持續寫入設計的儲存層，通常以串流或批次方式寫入，並依時間或標籤建立索引，可高效處理大量資料。

With a **relational database**, logs must be transformed into structured rows and inserted transactionally. As volume grows, performance is limited by transaction overhead, indexing, and locking behavior.

使用**關聯式資料庫**時，日誌需轉換為結構化資料列並進行交易式寫入。隨著資料量增加，寫入效能容易受到交易成本、索引維護與鎖定機制的限制。



## Scalability and Performance Characteristics | 擴展性與效能特性

Logging systems are built for horizontal scaling and can handle millions of events per second using batching, buffering, and asynchronous writes. They also support retention policies and efficient storage management.

日誌系統通常採水平擴展設計，透過批次寫入、緩衝與非同步機制，可支援每秒數百萬筆事件寫入，同時具備資料保留（Retention）與自動清理機制。

Relational databases face challenges under heavy logging workloads. Frequent inserts increase load on transaction logs and indexes, and scaling requires complex partitioning or sharding strategies.

關聯式資料庫在高頻寫入情境下會面臨瓶頸，例如交易日誌與索引負擔增加，擴展則需仰賴分區（Partition）或分片（Sharding）等較複雜策略。

Additionally, logging workloads are typically write-heavy, while relational databases are optimized for balanced read/write workloads.

此外，日誌負載通常是「寫多讀少」（分析前），而資料庫假設讀寫較均衡，兩者設計目標不一致。



## Flexibility of Data Structure | 資料結構彈性

Logging systems support **semi-structured or unstructured data**, such as JSON and dynamic fields, without requiring schema changes.

日誌系統可自然支援**半結構或非結構化資料**（例如 JSON），即使欄位變動也無需調整結構。

Relational databases require **predefined schemas**. Any change in log format requires schema updates or workarounds, which increases complexity over time.

關聯式資料庫需要 **預先定義 Schema**，一旦日誌格式改變，就需要進行資料庫結構調整或額外處理，長期可能造成維護負擔。



## Reliability and Failure Handling | 可靠性與錯誤處理

Logging pipelines achieve reliability through **buffering, retries, and queues** to avoid data loss during failures.

日誌系統透過 **緩衝（buffer）、重試（retry）與訊息佇列** 來確保在故障時減少資料遺失。

Relational databases rely on **ACID transactions**, which increase overhead and reduce efficiency for high-volume logging.

關聯式資料庫依賴 **ACID 交易保證**，但在高頻日誌寫入情境下會帶來較高延遲與效能成本。

**Operational Limitations | 維運上的限制**

| Requirement<br>需求         | DB Limitation<br>資料庫限制    |
| ------------------- | ---------------------------- |
| Log retention<br>日誌保留       | Hard (manual partition/drop)<br>困難（需手動分區或刪除） |
| Downsampling<br>降採樣        | Not native<br>無原生支援       |
| Hot/cold tier<br>熱 / 冷資料分層       | Not supported<br>不支援        |
| Streaming ingestion<br>串流寫入 | Weak<br>能力較弱       |


## Query and Analysis Model | 查詢與分析模型

Logging systems provide **search-oriented querying** optimized for time ranges and aggregations, suitable for monitoring and troubleshooting.

日誌系統提供 **搜尋導向** 的查詢方式，針對時間區間與聚合分析最佳化，適合監控與問題排查。

Relational databases use **SQL** for structured queries, but performance degrades with large-scale log data and text search.

關聯式資料庫使用 **SQL** 查詢結構化資料，但在大量日誌分析與全文搜尋時效率較差。



## Practical Recommendation | 實務建議

A **logging system should be the primary solution for raw logs**, ensuring scalability and observability.

建議使用**日誌系統作為原始日誌的主要儲存與分析平台**，以確保系統可擴展性與可觀測性。

A **relational database should be used only for derived or structured results**, such as aggregated metrics or business events.

而**關聯式資料庫適合作為輔助用途**，例如儲存彙總指標或業務事件。



## Comparison Table | 比較表

| Aspect<br>面向        | Logging System<br>日誌系統            | Relational DB<br>關聯式資料庫           |
| ------------- | ------------------------- | ----------------------- |
| Design Goal<br>設計目標   | High-throughput ingestion<br>高吞吐寫入 | Structured transactions<br>結構化交易 |
| Write Pattern<br>寫入模式 | Append-only<br>追加式               | Transactional<br>交易式           |
| Schema        | Flexible<br>彈性        | Fixed<br>固定                   |
| Scalability<br>擴展性   | Horizontal<br>水平擴展                | Limited<br>有限制                 |
| Query<br>查詢方式         | Search-oriented<br>搜尋導向           | SQL                     |
| Performance<br>效能重點   | Write + search<br>寫入＋搜尋       | Consistency<br>一致性             |
| Use Case<br>使用情境      | Logs / Observability<br>日誌 / 觀測      | Business data<br>商業資料           |


## Summary | 總結

The choice is not interchangeable:  
此兩種技術**不可互換使用**：  

* Use logging systems for logs and observability  
使用日誌系統處理日誌與監控  
* Use relational databases for structured business data  
使用關聯式資料庫處理結構化商業資料  

This separation ensures a scalable and maintainable architecture.  
透過明確分工，可打造更具擴展性與可維護性的系統架構。  

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Logging vs Relational DB in a Logging Pipeline © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌管線中的 Logging vs 關聯式資料庫 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際版](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

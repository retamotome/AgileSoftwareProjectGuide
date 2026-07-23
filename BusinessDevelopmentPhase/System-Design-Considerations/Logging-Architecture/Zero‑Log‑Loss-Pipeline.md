# Zero‑Log‑Loss Pipeline (Even During Crashes) | 日誌零遺失管線（即使發生當機）


![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Overview ｜ 概述

The objective of this architecture is to guarantee **no log loss** under failure scenarios such as container crashes, host failures, network interruptions, or collector restarts. The design ensures that logs are always persisted and recoverable at every stage of the pipeline.  
此架構的目標是在各種異常情境（如容器當機、主機故障、網路中斷、收集器重啟）下，確保**日誌零遺失**。設計上保證在每個處理階段都能進行持久化與復原。



## Recommended Architecture | 建議架構

```
[Application]
    ↓ (stdout / file)
[Docker JSON Log / File]
    ↓
[Fluent Bit (with disk buffer)]
    ↓
[Message Queue (Kafka with ACKs)]
    ↓
[Storage (Elasticsearch / Loki)]
```

This pipeline separates concerns into distinct layers: log generation, local persistence, reliable forwarding, centralized buffering, and long-term storage. Each layer contributes to durability and failure isolation.  
此管線將各項關注點拆分為不同層級：日誌產生、本地持久化、可靠傳輸、集中緩衝與長期儲存。每一層都強化了耐久性與錯誤隔離能力。



## Persistent Local Buffering | 本地持久化緩衝

To prevent data loss at the earliest stage, Fluent Bit is configured to store logs on disk before attempting to forward them. This ensures that logs are safely written to a persistent medium rather than held only in memory.  
為避免在最初階段發生資料遺失，Fluent Bit 會先將日誌寫入磁碟再進行傳送，確保資料儲存在持久媒介上，而非僅存在記憶體中。

**Example Configuration**  
**設定範例**

```ini
[SERVICE]
    storage.path              /var/log/fluentbit-buffer
    storage.sync              normal
    storage.checksum          on
    storage.backlog.mem_limit 50MB

[INPUT]
    Name tail
    Path /var/lib/docker/containers/*/*.log
    Tag docker.*
    storage.type filesystem
```

With this setup, logs are first written to the local filesystem. As a result, they remain available even if Fluent Bit crashes or the host system reboots. When the system recovers, Fluent Bit resumes forwarding from the persisted data without loss.  
透過此設定，日誌會先寫入本地檔案系統，因此即使 Fluent Bit 當機或系統重開機，日誌仍可保留。系統恢復後，Fluent Bit 可從既有資料繼續傳送，不會遺失。



## Reliable Delivery Through Acknowledged Messaging | 具確認機制的可靠傳輸（ACK）

After local buffering, logs are forwarded to a message queue that guarantees durability through acknowledgment mechanisms. Kafka is commonly used here because it ensures that data is replicated and confirmed before being considered successfully written.  
在本地緩衝之後，日誌會送至具確認機制的訊息佇列（Message Queue）。Kafka 常用於此，因為它會在資料完成複寫並確認後才視為寫入成功。

**Example Configuration**  
**設定範例**

```properties
acks=all
retries=∞
min.insync.replicas >= 2
```

In this configuration, a log entry is acknowledged only after it has been written to multiple replicas. This prevents data loss even if a broker fails, ensuring that logs are safely stored before the pipeline proceeds.  
在此設定中，日誌需寫入多個副本後才會被確認（ACK）。即使某個節點失效，也能避免資料遺失，確保日誌安全儲存後才繼續處理。



## Backpressure and Flow Control | 反壓處理與流量控制

When downstream systems such as Kafka or storage backends become unavailable or slow, the pipeline must handle the situation without dropping logs. This is achieved through layered buffering.  
當下游系統（如 Kafka 或儲存系統）不可用或效能下降時，管線必須能處理壅塞情況而不丟失日誌，這透過分層緩衝機制達成。

Fluent Bit temporarily stores logs on local disk, while Kafka provides centralized buffering once logs reach the queue. This coordinated buffering allows the system to absorb spikes or outages without losing data. Instead of discarding logs, the pipeline delays processing until downstream services recover.  
Fluent Bit 會先暫存於本地磁碟，而 Kafka 則在接收後提供集中式緩衝。這樣的分層設計可吸收流量尖峰或系統中斷，避免資料流失。管線會延後處理，而非丟棄日誌，直到下游恢復。



## Handling Container Removal Safely | 容器刪除時的安全處理

Even when containers are forcefully removed, logs remain protected because they are already captured and buffered earlier in the pipeline.  
即使容器被強制刪除，日誌仍能保護，因為已在前段流程完成擷取與緩衝。

For example:  
例如：

```bash
docker rm -f <container>
```

By continuously tailing container log files and persisting them locally, Fluent Bit ensures that logs are collected before the container disappears. As a result, container lifecycle events do not lead to log loss, preserving complete observability.  
透過持續 tail 容器日誌並寫入本地，Fluent Bit 能在容器消失前完成收集。因此容器生命週期變化不會造成日誌遺失，維持完整可觀測性。



## Crash Scenarios Coverage | 當機情境覆蓋

| Failure<br>故障情境            | Protected?<br>是否保護 | Why<br>原因                        |
| ------------------ | ---------- | ------------------------- |
| App crash<br>應用程式當機          | ✅          | Logs already written<br>日誌已寫出      |
| Container removed<br>容器被刪除  | ✅          | Fluent Bit captured<br>Fluent Bit 已擷取       |
| Fluent Bit crash<br>Fluent Bit 當機   | ✅          | Disk buffer<br>有磁碟緩衝               |
| Network down<br>網路中斷       | ✅          | Retry + buffer<br>重試 + 緩衝            |
| Kafka node failure<br>Kafka 節點故障 | ✅          | Replication<br>資料複寫               |
| Power loss<br>斷電          | ✅ (mostly)<br>（多數情況） | Depends on disk sync mode<br>取決於磁碟同步模式 |


---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Zero‑Log‑Loss Pipeline © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
日誌零遺失管線 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際版](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

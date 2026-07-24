# Agent-Based Collection vs Docker Log Driver | 代理式收集 vs Docker 記錄驅動


![Practical System Design Considerations](../../img/PracticalSystemDesignConsiderations.png)  

## Overview ｜ 概述

This section compares two common approaches for collecting container logs: using Docker’s built-in log drivers versus using an external agent (such as Fluent Bit) to tail log files. The focus is on reliability, failure handling, and suitability for production systems.  
本節比較兩種常見的容器日誌收集方式：使用 Docker 內建 log driver，以及使用外部代理（如 Fluent Bit）來進行日誌檔案 tail（持續讀取）。重點在於可靠性、異常處理能力與是否適合生產環境。



## Background and Problem Context | 背景與問題情境

By default, Docker stores container logs under:  
預設情況下，Docker 會將容器日誌儲存在：

```
/var/lib/docker/containers/
```

These log files can be removed when containers are deleted, depending on Docker configuration and cleanup policies. If logs are not collected in time, this behavior may lead to data loss.  
當容器被刪除時，這些日誌檔可能會被清除（取決於 Docker 設定與清理策略）。若日誌未即時收集，可能會導致資料遺失。

To address this, production systems typically adopt one of two approaches: direct log streaming via Docker log drivers or indirect collection through an external agent.  
為了解決此問題，生產系統通常採用兩種方式之一：透過 Docker log driver 直接串流日誌，或透過外部代理進行間接收集。



## Agent-Based Collection Approach | 代理式收集架構  

In this model, logs follow a multi-stage path:  
在此模型中，日誌會經過多個階段：

```
Application → stdout → Docker → Agent → Message Queue → Storage
```

The logging agent (e.g., Fluent Bit) reads logs from Docker-generated files and forwards them downstream. Because the agent continuously tails log files and can buffer data locally, logs are usually captured before containers are removed.  
日誌代理（例如 Fluent Bit）會從 Docker 產生的檔案中讀取日誌並轉送到下游系統。由於代理會持續 tail 檔案並支援本地緩衝，因此通常能在容器刪除前完成日誌擷取。

This approach emphasizes decoupling and reliability. Even if downstream systems fail, logs can be temporarily stored on disk and retried later, significantly reducing the chance of loss.  
此方法強調「解耦」與「可靠性」。即使下游系統故障，日誌仍可暫存在磁碟並於稍後重送，大幅降低資料遺失風險。

### Fluent Bit Tailing Approach | Fluent Bit 擷取模式

In this model, Docker still writes logs to its standard JSON files, and Fluent Bit reads them asynchronously:  
在此模式下，Docker 仍會將日誌寫入標準 JSON 檔案，而 Fluent Bit 以非同步方式讀取：

```
Docker → json.log → Fluent Bit tail → pipeline
```

This design separates log generation from log delivery. Docker only needs to write logs locally, while Fluent Bit handles forwarding independently.  
此設計將「日誌產生」與「日誌傳輸」分離。Docker 只需負責本地寫入，Fluent Bit 則獨立負責轉送。

Because Fluent Bit supports disk-based buffering and retry mechanisms, logs remain safe even if the pipeline is temporarily unavailable. Additionally, logs stay accessible on the host system, which simplifies debugging and operational troubleshooting.  
由於 Fluent Bit 支援磁碟緩衝與重試機制，即使管線短暫不可用，日誌仍可安全保留。此外，日誌仍存在於主機上，有助於除錯與維運排查。



## Docker Log Driver Approach | Docker Log Driver 架構

Docker provides built-in log drivers that allow containers to send logs directly to external systems without writing them to local files.  
Docker 提供內建 log driver，可讓容器直接將日誌送往外部系統，而不需寫入本地檔案。

**Example Configuration**  
**設定範例**

```bash
docker run \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224
```

With this configuration, logs are streamed directly from the container runtime to a logging backend (lower latency). This removes the need for file-based collection and simplifies the overall pipeline.  
透過此設定，日誌可直接從容器 runtime 傳送至後端（日誌系統），延遲較低，也不需經過檔案收集，整體流程更簡單。

However, because the container runtime is directly responsible for log delivery, it becomes tightly coupled with the availability and behavior of the logging system. If the backend is unavailable or slow, it can affect container execution or result in dropped logs due to limited buffering.  
但由於日誌傳輸由容器 runtime 直接負責，會與日誌系統高度耦合。若後端不可用或效能不佳，可能影響容器運作，或因緩衝不足而造成日誌遺失。



## Behavioral Differences and Trade-offs | 行為差異與取捨

The key difference between the two approaches lies in how they handle failures and system coupling.  
兩者最大的差異在於異常處理能力與系統耦合程度。

With Docker log drivers, log transmission is immediate but dependent on the target system. This can reduce latency but introduces risk if the destination is unavailable. Buffering is limited, and failures may propagate back to the container runtime.  
Docker log driver 可即時傳送日誌，但依賴目標系統狀態。雖然延遲低，但當目標不可用時風險較高，且緩衝能力有限，錯誤可能回傳影響容器。

With Fluent Bit tailing, there is a slight delay due to file-based collection, but reliability is significantly improved. Logs are first persisted locally, then forwarded with retry and buffering support. This makes the system more resilient to crashes, restarts, and network issues.  
Fluent Bit tailing 會因檔案收集產生些微延遲，但可靠性顯著提升。日誌會先寫入本地，再透過重試與緩衝機制轉送，使系統更能應對當機、重啟與網路問題。



## Comparison Overview | 比較總覽

| Aspect<br>面向                 | Docker Log Driver           | Fluent Bit Tailing           |
| ---------------------- | --------------------------- | ---------------------------- |
| Reliability<br>可靠性            | Moderate, backend-dependent<br>中等（依賴後端） | High, with local persistence<br>高（具本地持久化） |
| Crash resilience<br>當機韌性       | Limited buffering<br>緩衝能力有限           | Strong with disk buffering<br>磁碟緩衝強    |
| System coupling<br>系統耦合        | Tight integration<br>高耦合           | Loosely coupled<br>低耦合              |
| Local log availability<br>本地日誌保留 | Not retained locally<br>不保留        | Available on host<br>主機可存取            |
| Backpressure handling<br>反壓處理  | Weak<br>較弱                        | Strong<br>較強                       |
| Flexibility<br>彈性            | Limited routing options<br>路由選項有限     | Supports multiple outputs<br>支援多輸出    |
| Production suitability<br>生產環境適用性 | Situational<br>視情況而定                 | Strongly recommended<br>強烈建議         |


## Summary | 總結

Docker log drivers provide a simple and low-latency solution by streaming logs directly, but they introduce tight coupling and limited fault tolerance.  
Docker log driver 提供簡單且低延遲的解法，但會帶來高度耦合與較低的容錯能力。

In contrast, Fluent Bit tailing adds an extra layer but delivers significantly stronger reliability, buffering, and operational flexibility.  
相比之下，Fluent Bit tailing 雖增加一層架構，但能大幅提升可靠性、緩衝能力與維運彈性。

For production environments where log durability and failure handling are critical, the agent-based tailing approach is generally preferred.  
在重視日誌可靠性與異常處理的生產環境中，通常會優先選擇代理式（Agent-based）tailing 架構。


---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Agent-Based Collection vs Docker Log Driver © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
代理式收集 vs Docker 記錄驅動 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

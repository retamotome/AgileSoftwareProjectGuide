# Data Processing Strategy | 資料處理策略  

![Practical System Design Considerations](../img/PracticalSystemDesignConsiderations.png)  


## Data Format ｜ 資料格式

For configuration files, adopt **human-readable formats** such as JSON or YAML to simplify maintenance and reduce the need for complex queries.  
設定檔建議採用 **人類可讀格式**（如 JSON 或 YAML），以簡化維護並降低複雜查詢需求。  

When handling data that requires frequent searching, filtering, or analytical processing, leverage a **database system** to ensure optimal performance, scalability, and efficiency.  
對於需要頻繁搜尋、篩選或進行分析處理的資料，應使用 **資料庫系統**，以確保效能、可擴展性與處理效率。  


## Using External Storage ｜ 使用外部儲存體

Always perform a **full format** when initializing external storage (e.g., SD cards, HDDs, removable media).  
在初始化外部儲存（如 SD 卡、硬碟、可移除媒體）時，務必進行 **完整格式化（full format）** 。

- Full format scans and **identifies bad sectors**, marking them unusable.  
  完整格式化會掃描並標記 **壞軌（bad sectors）**  以避免使用。  

- Ensures **data integrity** and more reliable long-term operation.  
  確保 **資料完整性**，並提升長期運行的可靠性。  

> [!important]  
> **Avoid** using **shared folders or network-shared storage** for critical data:  
> **避免**將關鍵資料存放於 **共享資料夾或網路共享儲存**：  
> - Data transfer is **not encrypted by default** (e.g., SMB/NFS), unless explicitly configured.  
>   預設傳輸**未加密**（如 SMB/NFS），除非額外設定。  
> - Exposes risk of **data interception, leakage, or unauthorized access**.  
>   增加**資料攔截、外洩或未授權存取**風險。  


## Ensuring Reliable Data Sync ｜ 確保資料同步可靠性  

- **Closing a file (`close()`) is not enough**: It only flushes user‑space buffers to the OS; data may still remain in the kernel’s page cache.  
  **僅呼叫 `close()` 並不足夠**：僅會將使用者空間資料寫入 OS，資料仍可能停留在核心快取（page cache）。

- **Explicit flush required**: Use `fsync(fd)` or `fdatasync(fd)` on Linux/Unix, and `FlushFileBuffers(handle)` on Windows, to force both data and metadata to stable storage.  
  **需明確強制寫入**：在 Linux/Unix 使用 `fsync` / `fdatasync`，在 Windows 使用 `FlushFileBuffers`，確保資料與中繼資料真正寫入儲存裝置。  

- **Performance trade‑off**: Frequent sync calls can reduce throughput, so durability must be balanced with efficiency.  
  **效能取捨**：頻繁同步會降低效能，需在可靠性與效率間取得平衡。  

- **Structured persistence techniques**: Write‑ahead logging (WAL), checkpointing, and replication provide stronger guarantees of reliability and recovery compared to relying solely on sync calls.  
  **結構化持久化技術**：如 WAL（預寫式日誌）、Checkpoint、Replication，可提供更高的可靠性與復原能力。  

- **Best practice workflow**: `write → fsync/fdatasync → close` ensures data is truly committed to disk and minimizes risk of loss during crashes or power failures.  
  **最佳實務流程**：`write → fsync/fdatasync → close`，可確保資料真正寫入磁碟，降低當機或斷電風險。  


## Data Transmission ｜ 資料傳輸  

To ensure **data integrity, availability, and confidentiality** during transfer, it is essential to use secure protocols (e.g., **HTTPS, SFTP, TLS**) combined with strong encryption.  
為確保資料在傳輸過程中的**完整性、可用性與機密性**，應採用安全協定（如 **HTTPS、SFTP、TLS**）並搭配強式加密。  

These measures protect against tampering, interception, and unauthorized access.  
可有效防止資料遭到竄改、攔截或未授權存取。  


### Built-in Resume Capability ｜ 原生續傳能力

Built‑in resume capability means a protocol or system can **natively continue interrupted transfers** from the last completed point without external tools.  
原生續傳能力指協定或系統可在中斷後，直接從最後成功位置繼續傳輸，無需額外工具。  

Examples include:  
常見例子：

- **FTP** (via the `REST` command)  
  FTP（透過 `REST` 指令）
- **SFTP** (client‑side offset resume)  
  SFTP（客戶端續傳）
- **BitTorrent** (chunked transfer with checksums)  
  BitTorrent（分塊與校驗）
- **Cloud storage APIs** (e.g., Amazon S3 multipart uploads, Google Cloud Storage resumable uploads)  
  雲端儲存 API（如 S3 multipart、GCS 可續傳上傳）  

This feature is critical for large data or unstable networks, ensuring efficiency and saving bandwidth.  
此功能對於大型資料或不穩定網路環境非常重要，可確保效率並節省頻寬。  


### Paging / Chunking ｜ 分頁／分塊

For large data transfers, **paging or chunking** techniques improve reliability and memory management.  
對於大型資料傳輸，**分頁／分塊技術**可提升可靠性與記憶體管理。  

Data is broken into smaller segments, transferred sequentially, and reassembled at the destination.  
資料會被拆分為小區塊逐步傳輸，並於目的端重新組合。  

This approach:  
此方法優點：

- Simplifies error handling and retries (only failed chunks are resent).  
  僅需重新傳送失敗區塊，簡化錯誤處理。  

- Prevents excessive memory usage.  
  避免過度消耗記憶體。  

- Is widely adopted in **REST APIs** and cloud services for handling big datasets.  
  廣泛應用於 **REST API 與雲端服務**。  


### Limitations ｜ 限制

- Not all protocols support resume (e.g., **SCP, WebDAV**).  
  並非所有協定都支援續傳（如 **SCP、WebDAV**）。  

- Resume may fail if the source data changes mid‑transfer or if metadata (timestamps, permissions) is not preserved.  
  若來源資料於傳輸中變更或中繼資料（時間戳、權限）未保存，可能導致續傳失敗。  

- Peer or server availability is required (especially in BitTorrent).  
  需依賴節點或伺服器可用性（尤其是 BitTorrent）。    

---

# License｜授權條款

![BY NC SA](../../img/Cc-by-nc-sa.png)  
Data Processing Strategy © 2018 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
資料處理策略 © 2018 作者 潘貞元（Reta Pan），依 [姓名-非商業性-相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  
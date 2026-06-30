# Security Consideration | 安全性考量

![Practical System Design Considerations](../img/PracticalSystemDesignConsiderations.png)  

## Connections ｜ 連線安全  

All external connections **must be protected using strong encryption mechanisms** to ensure confidentiality, integrity, and authenticity of data in transit.  
所有外部連線**必須使用強式加密機制保護**，以確保傳輸資料的機密性、完整性與真實性。  

* Approved protocols:  
核准使用協定：  

  * **HTTPS (TLS 1.2 or above)**
  * **Secure WebSocket (WSS)**  

* **Plain-text communication protocols (e.g., HTTP, Telnet, FTP) are strictly prohibited**  
**禁止使用明文傳輸協定（如 HTTP、Telnet、FTP）**   
* TLS certificates must be:  
  TLS 憑證必須：  

  * Valid and issued by trusted authorities  
    由可信任憑證機構簽發且有效  

  * Properly stored and protected  
    妥善儲存並加以保護  

  * Periodically rotated  
    定期輪替更新


## Network Interface ｜ 網路介面

### Scope ｜ 範圍

The following interfaces are within scope of this control:  
以下介面皆屬本控制範圍：

* System network interfaces  
  系統網路介面
* Container network interfaces (e.g., Docker)  
  容器網路介面（如 Docker）
* REST APIs  
* Webhooks / callback endpoints  
  Webhook / 回呼端點  

### Network Access Control Requirements ｜ 網路存取控制要求

* All network ports must follow a **default-deny policy**:  
  所有網路連接埠必須採用**預設拒絕原則（default-deny）** ：

  * All ports are **denied by default**  
    預設全部禁止
  * Only explicitly required ports and services are **whitelisted**  
    僅允許明確需求的連接埠與服務（白名單）

* Unused or unnecessary services must be **disabled or removed**  
  未使用或不必要的服務必須停用或移除

* Implement **network segmentation** where applicable:  
  視情況實作網路區隔：

  * Internal vs external network isolation  
    內外網隔離
  * Container network isolation  
    容器網路隔離

* Administrative interfaces must:  
  管理介面必須：

  * Not be exposed to public networks  
    不可暴露於公開網路
  * Be restricted via VPN, bastion host, or internal network  
    需透過 VPN、跳板主機或內部網路限制存取


### Secure Communication Requirements ｜ 安全通訊要求

* All external and inter-service communication must be encrypted (TLS-based)  
  所有外部及服務間通訊必須使用 TLS 加密  
* Mutual TLS (**mTLS**) should be considered for **service-to-service communication**  
  服務間通訊建議使用雙向 TLS（mTLS）  
* Certificates and keys must be centrally managed (e.g., secret management system)  
  憑證與金鑰應集中管理（如 Secret 管理系統）  
* Avoid exposing services directly; use:  
  避免直接暴露服務，應使用：  

  * Reverse proxy (e.g., NGINX, Traefik)  
    反向代理（如 NGINX、Traefik）  
  * API Gateway where appropriate  
    API Gateway（視需求）  


## API Guard ｜ API 安全防護

### Scope ｜ 範圍

* All APIs exposed externally by the system  
  系統對外提供的所有 API
* All internal APIs between services/modules (including containerized services)  
  系統內部服務／模組間 API（含容器服務）  


### Input Validation and Threat Protection ｜ 輸入驗證與威脅防護

* All API requests must undergo **strict validation**, including:  
  所有 API 請求必須經過嚴格驗證，包括：

  * Data type, format, length, and range checks  
    資料型別、格式、長度與範圍檢查
  * Whitelisting of allowed parameters and values  
    參數與數值白名單

* The system must **reject immediately**:  
  系統必須立即拒絕：

  * Invalid commands  
  無效指令  
  * Malformed requests   
  格式錯誤請求  
  * Unexpected or suspicious input patterns  
  異常或可疑輸入  
* The system must explicitly protect against:  
  必須防範：

  * SQL Injection  
  SQL 注入  
  * Command Injection  
  指令注入  
  * Cross-Site Scripting (XSS)  
  跨站腳本攻擊  
  * Path Traversal  
  路徑穿越  
  * Deserialization vulnerabilities  
  反序列化漏洞  


### Authentication and Authorization ｜ 身分驗證與授權  

* All APIs must enforce **strong authentication**, such as:  
API 必須採用強式驗證：

  * OAuth 2.0
  * JWT (with signature validation)  
  JWT（需驗證簽章）

* Authorization must:  
  授權機制必須：

  * Follow the **principle of least privilege**  
    遵循最小權限原則
  * Enforce **role-based access control (RBAC)**  
    採用角色基礎存取控制（RBAC）

* Sensitive endpoints must include:  
  敏感端點需具備：

  * Token expiration and renewal  
    Token 過期與更新機制
  * Scope/permission validation  
    權限範圍驗證
  * Protection against token replay  
    防止 Token 重放攻擊

### API Protection Controls ｜ API 防護控制

* Implement **rate limiting / throttling** to prevent abuse and DoS attacks  
  實作流量限制防止濫用與 DoS

* Apply **IP allowlist/denylist** where appropriate  
  視需求使用 IP 白／黑名單

* Use **API Gateway or WAF (Web Application Firewall)** for:  
使用 API Gateway 或 WAF：

  * Traffic inspection  
  流量檢測  
  * Attack pattern detection  
  攻擊偵測  

* All API activities must be:  
API 行為需：

  * Logged (request, response status, source IP)  
  紀錄（請求、回應、來源 IP）  
  * Monitored for anomalies  
  進行異常監控  


### Secure Development and Operations (Recommended) ｜ 安全開發與維運（建議）  

* Enforce **API schema validation** (e.g., OpenAPI) during development and in CI  
強制 API Schema（如 OpenAPI）驗證  

* Prevent **hardcoded credentials** (use secret management)  
禁止硬編碼憑證（使用 Secret 管理）  

* Perform **security testing**:  
執行安全測試：  

  * Static analysis (SAST)  
  靜態分析  
  * Dependency scanning  
  套件弱點掃描  
  * API security testing  
  API 安全測試
 

## Database Security ｜ 資料庫安全  

### Use Stored Procedures ｜ 使用預存程序

Use **stored procedures** for all database operations instead of embedding raw SQL queries or data access logic directly in application code.  
所有資料庫操作應使用 **預存程序（Stored Procedures）** ，而非將原始 SQL 查詢或資料存取邏輯直接寫在應用程式中。  

Stored procedures help to:  
預存程序有助於：  

* Enforce consistent access control  
  強制一致的存取控制機制  
* Reduce the risk of SQL injection    
  降低 SQL 注入風險  
* Centralize business logic within the database layer   
  將商業邏輯集中於資料庫層  
* Improve maintainability and auditability    
  提升系統維護性與稽核能力  

Direct queries (especially dynamic SQL constructed in code) should be avoided unless absolutely necessary and properly parameterized.  
應避免直接使用查詢（特別是於程式中動態組合 SQL），除非確實必要且已妥善參數化處理。  

### Password Storage and Encryption ｜ 密碼儲存與加密

Passwords **must never be stored in plain text**. They must be securely protected using **strong cryptographic hashing algorithms**, such as:  
密碼**絕不可明文儲存**，必須透過**強式加密雜湊演算法**進行保護，例如：  

* **bcrypt**
* **Argon2**
* **PBKDF2**

> [!important]  
> Algorithms like **MD5** or **SHA-1 are no longer considered secure** and must not be used for password storage.  
> 像 **MD5** 或 **SHA-1 已不再被視為安全** 的演算法，禁止用於密碼儲存。

Additional requirements:  
其他要求：  

* Always use a **unique salt** for each password  
每個密碼必須使用 **唯一 Salt** 值  
* Consider using **adaptive hashing** (configurable workload/cost factor)  
建議使用 **可調整強度的雜湊方式**  
* Do not implement your own cryptographic algorithms—use well-established libraries  
不可自行實作加密演算法，應使用成熟且經驗證的函式庫  


## Removable Devices ｜ 可移除裝置  

All external interfaces supporting removable devices (e.g., USB ports, SD card slots, external storage interfaces) shall be **disabled or strictly controlled** in systems delivered to customers, in order to mitigate cybersecurity risks and prevent unauthorized access or unintended system modification.  
所有支援可移除裝置（如 USB、SD 卡、外接儲存）的外部介面，在交付客戶的系統中均應 **停用或嚴格控管** ，以降低資安風險並防止未授權存取或系統被異常修改。  

In alignment with **IEC 61508 (functional safety lifecycle and system integrity)** and **ISO 13849 (safety-related parts of control systems)**, the following requirements shall apply:  
依據 **IEC 61508（功能安全生命週期與系統完整性）**  與 **ISO 13849（控制系統安全相關部分）** ，應符合以下要求：  

### Requirements ｜ 要求  

* **Default State (Deployment) ｜ 預設狀態（部署）** 

  * All removable device interfaces shall be **disabled by default** in production/released systems.  
  所有可移除裝置介面在正式環境中必須 **預設停用**  
  * Access shall only be permitted through **authorized and controlled service procedures**.  
  僅允許透過 **授權且受控的維護流程** 進行操作  

* **Security and Safety Justification ｜ 安全與功能安全理由**

  * Removable media interfaces shall be treated as **potential hazard injection points**, capable of introducing:  
  可移除媒體應視為 **潛在風險注入點**，可能引入：  

    * Unauthorized software  
    未授權軟體  
    * Malware or configuration changes  
    惡意程式或設定變更  
    * Unvalidated firmware updates  
    未驗證韌體更新  

  * Disabling such interfaces supports:  
  停用這些介面有助於：  
 
    * Preservation of **system integrity**  
    維持系統完整性  
    * Prevention of **unauthorized safety function bypass**  
    防止安全功能被繞過  
    * Compliance with **defense-in-depth principles**  
    符合縱深防禦原則


* **Controlled Enablement ([Maintenance Mode](./System-Mode-and-State.md#system-modes--系統模式)) ｜ 受控啟用（[維護模式](./System-Mode-and-State.md#system-modes--系統模式)）**   

  * If required for diagnostics or maintenance:  
  若需診斷或維護：  

    * Interfaces shall only be enabled in a **restricted system Mode** (e.g., *Maintenance Mode*)  
    僅能於 **受限系統模式（如維護模式）**  中啟用  
    * Access shall require:  
    存取必須具備：  

      * **Authentication and authorization**  
      驗證與授權  
      * **Audit logging**  
      稽核紀錄  
      * **Physical or procedural control (e.g., service personnel only)**  
      實體或流程控管（限維修人員）  

* **Traceability and Verification ｜ 可追溯性與驗證**  
  * The status (enabled/disabled) of removable interfaces shall be:  
  介面狀態（啟用／停用）必須：  
    * Defined in the **Safety Requirements Specification (SRS)**  
    定義於 **SRS（安全需求規格）**  
    * Verified during **validation and acceptance testing**  
    於驗證與驗收測試中確認  
    * Traceable to **hazard analysis and risk assessment** (e.g., FMEA, threat modeling)  
    可追溯至風險分析（如 FMEA、威脅建模）  

* **Failure and Fault Handling ｜ 故障與異常處理**
  * Any unintended activation or misuse of removable interfaces shall:  
  若發生未預期啟用或誤用：  
    * Trigger **diagnostic detection mechanisms**  
    啟動診斷機制  
    * Cause transition to a **safe or risk-reduced state**, if system integrity is compromised  
    若影響系統完整性，需轉入 **安全或降風險狀態**



### Compact Architecture Rule ｜ 架構簡則

> Removable device interfaces (e.g., USB ports) shall be disabled in all production systems. Any temporary enablement shall be restricted to authorized maintenance modes with proper authentication, monitoring, and traceability, ensuring system integrity and compliance with functional safety requirements.   
> 生產系統中所有可移除裝置介面（如 USB）應停用；如需暫時啟用，僅能在授權的維護模式下進行，並需具備驗證、監控與可追溯機制，以確保系統完整性並符合功能安全要求。


## Third-Party Utility Management ｜ 第三方元件管理

* All third-party utilities used in this project must be thoroughly reviewed and scanned to identify any potential vulnerabilities or backdoor risks. Identified utilities should then be documented in the Software Bill of Materials (SBOM).   
本專案使用的所有第三方工具必須經過完整審查與弱點掃描，以識別潛在漏洞或後門風險，並納入 SBOM（軟體物料清單）中管理。  
* The SBOM should:   
SBOM 應包含：  
    * List all dependencies, including both direct and transitive   
    列出所有相依元件（含直接與間接相依）  
    * Record each dependency's version, license, source, and hash   
    記錄每個元件的版本、授權、來源與雜湊值

---

# License｜授權條款

![BY NC ND](../../img/Cc-by-nc-sa.png)  
Security Consideration © 2018 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
安全性考量 © 2018 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

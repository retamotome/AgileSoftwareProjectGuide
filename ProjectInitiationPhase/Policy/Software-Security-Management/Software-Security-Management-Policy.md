# 軟體安全管理政策 / Software Security Management Policy

**文件編號：** ISMS-SEC-DEV-001  
**Document ID:** ISMS-SEC-DEV-001  

**版本：** v1.0  
**Version:** v1.0  

**生效日期：** YYYY/MM/DD  
**Effective Date:** YYYY/MM/DD  

**文件等級：** 內部文件  
**Document Classification:** Internal Document

***

## **目的（Purpose）** 

為確保公司於軟體開發生命週期中之資訊安全，降低供應鏈攻擊、未授權軟體使用及憑證外洩等風險，建立一致性與可稽核之安全控管機制，並符合 ISO/IEC 27001:2022 標準要求，特訂定本政策。  
To ensure information security throughout the software development lifecycle, reduce risks such as supply chain attacks, unauthorized software usage, and credential leakage, and establish consistent and auditable security controls in compliance with ISO/IEC 27001:2022 requirements, this policy is hereby established.

***

## **適用範圍（Scope）** 

本政策適用於所有涉及軟體開發、測試、建置及部署之相關活動，包含公司內部人員、外包人員及合作夥伴於公司開發環境中之操作行為。  
This policy applies to all activities related to software development, testing, build, and deployment, including operations performed by internal employees, outsourced personnel, and partners within the company’s development environment.

***

## **角色與責任（Roles and Responsibilities）** 

| 角色 / Role | 職責 / Responsibilities |
| --------- | ----------------------------- |
| 開發人員 / Developers | 遵循安全開發規範與套件管理要求，確保程式碼與依賴元件安全性 / Follow secure development standards and package management requirements to ensure the security of code and dependency components |
| 資訊處 / IT Department | 負責軟體審核、風險監控與資安事件處理 / Responsible for software review, risk monitoring, and information security incident handling |
| 部門主管 / Department Managers | 督導政策執行及落實情形 / Supervise policy implementation and enforcement |
| ISMS 管理代表 / ISMS Management Representative | 負責政策維護、更新及對應內外部稽核 / Responsible for policy maintenance, updates, and coordination with internal and external audits |

***

## **政策內容（Policy Statements）** 

### **軟體與套件管理（Software and Package Management）** 

**控制要求（Control Requirements）** 

* 開發所使用之軟體、工具、套件與服務須經資訊處審核並核准後方可使用  
  Software, tools, packages, and services used for development must be reviewed and approved by the IT Department before use.
* 僅允許使用官方來源或經驗證之可信發布者  
  Only official sources or verified trusted publishers are allowed.
* 使用之軟體須具備安全政策與隱私保護聲明  
  Software in use must provide security policies and privacy statements.
* 優先選用具備國際資安認證之產品（如 ISO 27001、SOC 2）  
  Prefer products with international security certifications (e.g., ISO 27001, SOC 2).
* 未列入核准清單之軟體，須完成正式審查與風險評估程序  
  Software not on the approved list must undergo formal review and risk assessment.

**ISO 控制對應（ISO Control Mapping）** 

* A.8.9 Configuration Management
* A.8.19 Installation of Software on Operational Systems
* A.5.9 Inventory of Information and Other Associated Assets
* A.5.31 Legal, Statutory, Regulatory and Contractual Requirements

***

### **開發依賴與安裝安全（Dependency and Installation Security）** 

**控制要求（Control Requirements）** 

* 所有專案須採用版本鎖定機制（Lock File）  
  All projects must adopt version-locking mechanisms (Lock File).
* 禁止安裝未經驗證之最新版本套件  
  Installation of unverified latest package versions is prohibited.
* 套件安裝過程不得出現異常行為（如非預期網路連線）  
  No abnormal behavior is allowed during package installation (e.g., unexpected network connections).
* 應使用安全安裝方式（例如 npm ci、pip lock）  
  Secure installation methods should be used (e.g., `npm ci`, `pip lock`).
* 預設應停用第三方自動執行腳本機制  
  Third-party auto-executed script mechanisms should be disabled by default.

**ISO 控制對應（ISO Control Mapping）** 

* A.8.28 Secure Coding
* A.8.25 Secure Development Lifecycle
* A.8.23 Web Filtering
* A.8.7 Protection Against Malware

***

### **敏感資料保護（Sensitive Data Protection）** 

**控制要求（Control Requirements）** 

* 禁止於本機儲存敏感資訊（如密碼、金鑰、憑證、Token）  
  Storing sensitive information locally is prohibited (e.g., passwords, keys, certificates, tokens).
* 憑證應透過安全機制管理（如環境變數或秘密管理系統）  
  Credentials must be managed through secure mechanisms (e.g., environment variables or secret management systems).
* 應使用短效或動態憑證以降低風險  
  Short-lived or dynamic credentials should be used to reduce risk.
* 禁止使用本機預設憑證儲存路徑（如 `~/.aws`、`~/.ssh` 等）  
  Use of default local credential storage paths is prohibited (e.g., `~/.aws`, `~/.ssh`).

**ISO 控制對應（ISO Control Mapping）** 

* A.8.12 Data Leakage Prevention
* A.5.34 Privacy and Protection of PII
* A.8.2 Privileged Access Rights
* A.5.15 Access Control

***

### **開發環境隔離（Development Environment Isolation）** 

**控制要求（Control Requirements）** 

* 開發作業須於隔離環境中進行（如 Container、VM 或 Sandbox）  
  Development activities must be conducted in isolated environments (e.g., containers, VMs, or sandboxes).
* 禁止於宿主系統（Host）直接執行未經控管之程式  
  Direct execution of uncontrolled programs on host systems is prohibited.
* 開發、測試與佈署環境須明確區隔  
  Development, testing, and deployment environments must be clearly separated.

**ISO 控制對應（ISO Control Mapping）** 

* A.8.31 Separation of Development, Test and Production
* A.8.20 Network Security
* A.5.23 Information Security for Use of Cloud Services

***

### **安全測試與掃描（Security Testing and Scanning）** 

**控制要求（Control Requirements）** 

* 程式碼在 Commit 與 Pull Request 前須執行安全掃描  
  Security scanning must be performed before commits and pull requests.
* 每次版本發布前須完成完整安全檢測  
  Complete security testing must be completed before each release.
* CI/CD 流程應整合自動化安全檢查機制（如 SCA、Secret Scan）  
  CI/CD pipelines should integrate automated security checks (e.g., SCA, secret scanning).

**ISO 控制對應（ISO Control Mapping）** 

* A.8.29 Security Testing in Development
* A.8.8 Management of Technical Vulnerabilities
* A.8.16 Monitoring Activities

***

### **資安事件應變（Information Security Incident Response）** 

**控制要求（Control Requirements）**   
當發現或懷疑開發環境遭受攻擊時，應立即執行以下措施：  
When an attack on the development environment is detected or suspected, the following actions shall be taken immediately:

* 停止使用受影響之系統或工具  
  Stop using affected systems or tools.
* 立即隔離網路連線  
  Immediately isolate network connections.
* 通報主管及資訊處  
  Notify department managers and the IT Department.
* 撤銷所有存取憑證與 Token  
  Revoke all access credentials and tokens.
* 更換相關帳號密碼與金鑰  
  Rotate related account passwords and keys.
* 重建乾淨執行環境  
  Rebuild a clean execution environment.
* 檢查是否影響 CI/CD、版本與產品發布  
  Verify impacts on CI/CD, versioning, and product releases.

**ISO 控制對應（ISO Control Mapping）** 

* A.5.24 Information Security Incident Management Planning
* A.5.25 Assessment and Decision on Information Security Events
* A.5.26 Response to Information Security Incidents
* A.5.27 Learning from Information Security Incidents

***

### **安全認知與訓練（Security Awareness and Training）** 

**控制要求（Control Requirements）** 

* 人員須具備供應鏈攻擊與安全開發之基本認知  
  Personnel must have basic awareness of supply chain attacks and secure development.
* 應定期辦理資訊安全教育訓練  
  Information security education and training should be conducted regularly.
* 應強化對常見錯誤觀念之認知修正（如過度信任開源套件）  
  Awareness correction for common misconceptions should be reinforced (e.g., overtrust in open-source packages).

**ISO 控制對應（ISO Control Mapping）** 

* A.6.3 Information Security Awareness, Education and Training

***

## **控制要求與稽核證據（Control Requirements and Audit Evidence）** 

### **安全控制與證據對應（Security Control and Evidence Mapping）** 

| 控制編號 / Control ID | 控制項目 / Control Item | 控制要求 / Control Requirement | ISO 對應 / ISO Mapping | 稽核證據 / Audit Evidence | 責任單位 / Responsible Unit |
| ----- | ----------- | ----------------- | ------ | ------------------- | ------ |
| SC-01 | 軟體白名單管理 / Software Whitelist Management | 所有工具須經核准 / All tools must be approved | A.8.19 | 軟體清冊、核准紀錄 / Software inventory and approval records | 資訊處 / 研發單位 / IT Dept. / R&D |
| SC-02 | 套件來源驗證 / Package Source Verification | 僅允許官方來源 / Only official sources allowed | A.8.7 | SCA 掃描報告 / SCA scan reports | 研發單位 / R&D  |
| SC-03 | 套件版本鎖定 / Package Version Locking | 使用 lock file / Use lock files | A.8.25 | 原始碼版本庫 / Source code repository | 研發單位 / R&D  |
| SC-04 | 安裝安全控制 / Installation Security Control | 禁止執行 scripts / Disable script execution | A.8.28 | 設定截圖或設定檔 / Configuration screenshots or files | 研發單位 / R&D  |
| SC-05 | 憑證保護 / Credential Protection | 禁止本機存放 / No local storage | A.8.12 | Secret 掃描報告 / Secret scan reports | 資訊處 / IT Dept. |
| SC-06 | 環境隔離 / Environment Isolation | 使用 VM / Container / Use VM / Container | A.8.31 | 環境設定文件 / Environment configuration documents | 研發單位 / R&D  |
| SC-07 | 軟體成分分析（SCA） / Software Composition Analysis (SCA) | 使用安全掃描工具 / Use security scanning tools | A.8.29 | Snyk / BlackDuck 報告 / Snyk / BlackDuck reports | 研發單位 / R&D  |
| SC-08 | CI/CD 安全控制 / CI/CD Security Control | 實施 security gate / Enforce security gate | A.8.16 | Pipeline 紀錄 / Pipeline records | 研發單位 / R&D  |
| SC-09 | 事件應變 / Incident Response | 執行通報流程 / Execute notification process | A.5.26 | 事件通報紀錄 / Incident notification records | 資訊處 / IT Dept. |
| SC-10 | 發布安全 / Release Security | 發布前完成掃描 / Complete scanning before release | A.8.8 | Release 安全報告 / Release security reports | 研發單位 / R&D  |

***

## **風險說明（Risk Considerations）** 

本政策主要因應以下資訊安全風險：  
This policy primarily addresses the following information security risks:

* 開源套件供應鏈攻擊 / Open-source package supply chain attacks
* 惡意第三方程式執行 / Execution of malicious third-party code
* 憑證外洩及存取權限濫用 / Credential leakage and access privilege abuse
* CI/CD 流程遭植入惡意程式 / Malicious code injection into CI/CD pipelines

***

## **稽核與遵循（Compliance and Audit）** 

* 本政策納入公司資訊安全管理制度（ISMS）稽核範圍  
  This policy is included in the company’s ISMS audit scope.
* 各研發單位須提供以下稽核證據：  
  Each R&D unit must provide the following audit evidence:
  * 套件來源與使用清單 / Package source and usage list
  * 安全掃描與測試報告 / Security scan and test reports
* 若發現不符合事項，應依規定提出矯正與預防措施（CAPA）  
  If nonconformities are identified, corrective and preventive actions (CAPA) must be submitted per requirements.
* 資訊處與研發單位應定期執行抽查與監控  
  The IT Department and R&D units should conduct regular spot checks and monitoring.

***

## **文件控管（Document Control）** 

| 項目 / Item | 說明 / Description |
| ------ | ------------ |
| 文件擁有單位 / Document Owner |  |
| 審核單位 / Reviewing  | 資訊處 / IT Department |
| 核准單位 / Approving  | 管理階層 / Management |
| 審查週期 / Review Cycle | 每年至少一次或重大變更時 / At least annually or upon major changes |

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)     
Software Security Management Policy © 2026 by Jen Yuan Pan is licensed under Attribution-NonCommercial-ShareAlike 4.0 International.  
軟體安全管理政策 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  


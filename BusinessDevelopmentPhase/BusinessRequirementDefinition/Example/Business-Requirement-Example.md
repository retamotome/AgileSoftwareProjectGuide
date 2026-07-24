# Business Requirement Example ｜ 商業需求範例

## BR-001: Secure User Login ｜ BR-001：強化使用者登入安全

### Document Information ｜ 文件資訊

| Field (欄位)       | Value (說明)                  | 
| ------------ | ----------------------- |
| Owner (負責人)       | Product Manager (產品經理)        | 
| Stakeholders (利害關係人) | IT, Security, End Users (IT、安全部門、使用者) |
| Priority (優先級)     | High (高)                   | 
| Status (狀態)       | Approved (已核准)                |


### Business Objective ｜ 商業目標  

The system shall improve user authentication security and reduce login-related failures.  
系統應提升使用者驗證安全性並降低登入相關失敗率。  


### Problem Statement ｜ 問題描述

Users frequently encounter login failures due to unclear error messages and inconsistent validation.  
使用者因錯誤訊息不明確及驗證機制不一致，經常發生登入失敗問題。  


### Scope ｜ 範圍
> Define what is included and excluded.  
> 定義包含與不包含的範圍。  

#### In Scope ｜ 範圍內

* User login via username/password  
  使用帳號／密碼登入
* Authentication validation  
  身分驗證機制
* Error handling  
  錯誤處理機制

#### Out of Scope ｜ 範圍外

* Password reset  
  密碼重設
* Multi-factor authentication (Phase 2)  
  多重驗證（第二階段）


### Capabilities ｜ 性能

| Item (項目)                | Description (說明)                              | 
| ------------------- | ----------------------------------------- | 
| User Authentication (使用者驗證) | System shall authenticate users securely (系統應使用安全性驗證)  | 
| Error Feedback (錯誤的回應訊息)      | System shall provide clear login feedback (系統應提供清楚的回應訊息) | 


### Stakeholder Needs ｜ 利害關係人需求

| Stakeholder (利害關係人)  | Need (需求)              | Priority (優先級) |
| ------------- | ----------------- | -------- |
| End User (使用者)      | Easy login (容易登入)        | High (高)     |
| Security Team (資安團隊) | Strong validation (強化驗證機制) | High (高)     |


### Acceptance Criteria ｜ 驗收條件

* The system shall allow authorized users to access the system securely  
  系統應允許授權使用者安全存取系統  

* The system shall prevent unauthorized access  
  系統應防止未授權存取  

* The system shall provide clear feedback on login failure  
  系統應於登入失敗時提供明確的回應訊息  

### Success Metrics ｜ 成效指標

| Metric (指標)                 | Target (目標)         |
| ---------------------- | -------------- | 
| Login success rate (登入成功率)     | ≥ 95%          |
| Failed login reduction (登入失敗降低比例) | -30%           |
| Security compliance (安全合規)   | 100% compliant (100% 符合) |

### General Considerations ｜ 通用考量
> Please refer to [General Considerations](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/material/GeneralConsiderations.md) for details.  
> 詳細內容請參考[通用考量項目](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/material/GeneralConsiderations.md)  

#### Non-Functional Requirements ｜ 非功能性需求

| Category (分類)     | Requirement (需求說明)             |
| ------------ | ----------------------- | 
| Performance (效能)  | Response < 2 sec (回應時間 < 2 秒)       |
| Security (安全性)     | Authentication required (必須進行身分驗證) |
| Availability (可用性) | 99.9% uptime (99.9% 系統可用率)            |
| Scalability (擴展性)  | Support 1000+ users (支援 1000+ 使用者)    |


#### Constraints & Compliance ｜ 限制與合規

| Type (類型)      | Description (說明)                          |
| --------- | ------------------------------------ | 
| Technical (技術限制) | Must integrate with LDAP (必須整合 LDAP)             |
| Legal (法規要求)     | GDPR compliance (符合 GDPR)                      |
| Security (安全要求)  | Password encryption required (密碼需加密儲存)         |


#### Assumptions & Dependencies ｜ 假設與相依性

| Type (類型)      | Description (說明)                          |
| --------- | ------------------------------------ | 
| Assumption (假設條件) | Assumed condition (預設成立之條件)   |
| Dependency (相依項目) | External dependency (外部系統或服務依賴) | 

---

# License｜授權條款

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Business Requirement Example © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
商業需求範例 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  



# **軟體安全管理施行細則**

***

## **目的**

本文件用於規範軟體開發過程中之資訊安全實務作業，確保開發環境、套件使用與部署流程符合公司資安要求，並有效降低供應鏈攻擊、惡意套件及憑證外洩風險。

***

## **適用範圍**

適用於所有參與軟體開發、測試、建置與部署之人員，包含內部員工、外包人員與合作夥伴。

***

## **開發環境安全管理**

### **軟體與套件使用規範**

所有開發相關軟體、工具、套件與服務**得**符合以下原則：

* 僅可使用經資訊部門核准之軟體版本
* 套件來源必須為官方發布（官方封閉佈署的 repository，如 Ubuntu）
* 應具備明確之安全政策與隱私政策文件
* 優先選用具備國際資安認證之產品（如 ISO 27001、SOC 2）
* 開發工具之擴充套件（Extension / Plugin）須為官方發布版本

未列於核准清單之軟體、工具、套件與服務，**得**完成審查程序，並提供：

* 使用授權條款（License）
* 服務條款（Terms of Use / Service）
* 隱私政策（Privacy Policy）
* 第三方授權聲明（如有第三方依賴項目）

***

### **依賴套件與安裝安全**

#### **標準操作流程**

1. 初始化專案時，依據資訊部門核准之軟體版本，立即建立版本鎖定檔（Lock file）
2. 僅允許透過鎖定檔安裝套件
3. 禁止直接安裝最新版本

#### **建議實務**

**Node.js**

```bash
npm config set ignore-scripts true
npm ci
```

**Python**

```bash
pip install -r requirements.lock
```

#### **安全檢查要求**

* 安裝過程不得出現異常 log
* 不得產生非預期對外連線
* 若出現以下情況，應立即停止：
  * install script 嘗試執行 shell 指令
  * 下載未知來源檔案
  * 存取系統敏感路徑

***

### **敏感資料管理**

應遵循「不落地存放」原則：

* 禁止於**本機**儲存任何密碼、金鑰、Token 或憑證
* 禁止於**程式內**寫入任何密碼、金鑰、Token 或憑證
* 必須使用環境變數或秘密管理系統
* 優先採用短效憑證（temporary credential）

禁止使用以下預設憑證位置：

* `~/.aws`
* `~/.kube`
* `~/.ssh`

***

### **開發環境隔離**

所有開發作業須於隔離環境中執行：

* 建議使用 Docker Container 或虛擬機（VM）
* 禁止在主機 OS 直接操作未隔離套件
* Python 開發需使用虛擬環境：

```bash
python -m venv .venv
```

***

### **安全檢測與發布控管**

#### **標準開發流程（DevSecOps）**

```
需求 → 設計 → 開發 → Commit → PR → CI → 安全檢查 → Release
                ↓        ↓        ↓
              SCA    Secret Scan   Security Gate
```
SCA: Software Composition Analysis 軟體組成分析  

***

#### **必要檢查項目**

| 類型         | 工具建議                   |
| ---------- | ---------------------- |
| 套件風險（SCA）  | Snyk / BlackDuck       |
| Secrets 掃描 | GitGuardian            |
| 程式碼分析      | SonarQube              |
| 開源依賴       | OWASP Dependency-Check |

***

#### **Release 阻擋條件**

以下情況**不得**發布：

* 存在 High / Critical 漏洞
* 出現未授權套件
* 發現憑證外洩
* 安全掃描未完成

***

#### **CI/CD 控制範例**

```yaml
- name: Security Scan
  run: snyk test --severity-threshold=high

- name: Block Risk Build
  if: failure()
  run: exit 1
```

***

## **OT / EFEM / 機械手臂安全控制**

針對設備控制環境，須額外實施以下控管：

| 控制項目    | 控制要求                 | 實務說明   |
| ------- | -------------------- | ------ |
| 設備網路隔離  | EFEM 與 IT 網路分離       | 防止橫向滲透 |
| 控制程式完整性 | 程式需簽章或 hash 驗證       | 防止篡改   |
| 外部媒體管控  | 禁止 USB 使用            | 防止惡意帶入 |
| 套件白名單   | OT 環境使用 offline repo | 避免外部下載 |
| 權限控管    | Robot / PLC 限權       | 防止濫用   |
| 即時監控    | 建立異常行為警示             | 提升偵測能力 |

***

### **關鍵風險提醒**

* 不可直接將 CI/CD 結果部署至機台
* 必須經過模擬或 staging 
* 所有控制程式需可追溯版本

***

## **資安事件應變流程**

當發現異常時，必須立即依以下步驟執行：

### **立即處置**

* 停止使用開發環境與工具
* 停止所有套件安裝與 CI 任務

### **隔離系統**

* 中斷所有網路連線（Wi-Fi / LAN）

### **通報處理**

* 通報直屬主管與資訊部門

### **帳號與憑證處理**

* 撤銷所有 Token（Git、API、Cloud）
* 更換所有帳密與金鑰

### **環境復原**

* 重新安裝系統或建立新環境（VM）

### **影響分析**

* 檢查最近 commit
* 檢查 CI/CD pipeline
* 確認是否已影響產品或客戶

***

## **安全風險認知**

### **觀念說明**

* 安裝套件等同執行第三方程式
* 內網環境仍可能透過套件來源遭入侵
* Code Review 軟體無法防範供應鏈攻擊

***

### **供應鏈攻擊重點說明**

供應鏈攻擊通常透過合法套件進入系統，其特性包括：

* 利用開發流程執行惡意程式
* 將後門植入正式版本
* 影響範圍擴及公司與客戶

參考資訊:  
* [從 NPM 憑證竊取到 GitHub Actions 惡意腳本：剖析 CI/CD 流水線供應鏈攻擊手法與防禦策略](https://www.dreamjtech.com/25250/)   
* [Postmortem: TanStack npm supply-chain compromise](https://tanstack.com/blog/npm-supply-chain-compromise-postmortem)  
* [Supply Chain Attack Affecting Numerous npm and PyPI Packages](https://digital.nhs.uk/cyber-alerts/2026/cc-4781)  
* [New Shai-Hulud malware wave compromises 600 npm packages](https://www.bleepingcomputer.com/news/security/new-shai-hulud-malware-wave-compromises-600-npm-packages/)



***

## **遵循與管理要求**

* 所有人員需確實遵循本施行細則
* 違反規範將依公司制度進行處理
* 資訊部門有權進行稽核與抽查

***

---

## License｜授權條款

![BY NC ND](../../../img/Cc-by-nc-sa.png)     
軟體安全管理施行細則 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  


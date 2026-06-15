# Business Development Phase ｜ 商業開發階段

## Overview ｜ 概述

This phase focuses on defining clear, testable business requirements and preparing a foundation for behavior-driven validation.  
本階段著重於定義**清楚且可測試的商業需求**，並建立行為驅動驗證（Behavior-Driven Validation）的基礎。  

Teams should establish acceptance criteria early and use Behavior-Driven Development (BDD) practices to keep requirements aligned with business goals.  
團隊應及早建立驗收標準（Acceptance Criteria），並採用行為驅動開發（BDD）實務，以確保需求與商業目標保持一致。    

> [!important]  
> * **Business focus only**: The Business Development Phase deliberately excludes technical design or implementation details.  
  **僅聚焦商業面**：本階段刻意排除技術設計與實作細節。  
> * **Requirements clarity**: Documentation should emphasize business needs, expected outcomes, and acceptance criteria, not technical specifics.  
  **需求清晰化**：文件應強調商業需求、預期成果與驗收標準，而非技術細節。  
> * **Accessible criteria**: Acceptance criteria must be easily translatable into test cases without requiring technical knowledge.  
  **易於轉換的驗收標準**：驗收條件應可在不具備技術背景的情況下，輕鬆轉換為測試案例。  
> * **Define “what” and “why”**: Teams should describe desired outcomes and business value, avoiding “how” the solution will be implemented.  
  **定義「做什麼」與「為什麼」**：團隊應描述期望成果與商業價值，避免說明「如何實作」。  
> * **Flexibility preserved**: Keeping requirements non-technical ensures alignment with stakeholders and allows freedom in exploring technical solutions later.  
  **保留彈性**：維持需求的非技術性可確保與利害關係人對齊，並讓後續技術解法保有彈性。  


## Prerequisites ｜ 前置條件

For more background, see:  
更多背景知識請參考：
- [Lesson 01: Agile and Software Engineering 敏捷與軟體工程全覽](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-11-09_AgileProjectMgt/README.md)
- [Lesson 02: Business Requirement Definition 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md)


## Guidance ｜ 指引

+ [Writing Requirements Effectively 撰寫需求的高效方法](./BusinessRequirementDefinition/Writing-Requirements-Effectively.md).  
+ [Practical System Design Considerations 系統設計實務考量](./System-Design-Considerations/README.md).    

<details>
<summary>News and Reference｜參考資訊</summary>

+ 2026-06-09 [被黃仁勳酸「自以為上帝」！OpenAI 叛將阿莫迪，如何成為改寫 AI 規則的「超級領跑者」？](https://www.managertoday.com.tw/articles/view/72245)   

</details>

## Key Expected Deliverables ｜ 主要交付成果

* Software Engineering:  
  軟體工程：  

  * Well-defined business and system requirements.  
    明確定義的商業與系統需求  
  * A virtualized system framework for early validation and collaboration.  
    用於早期驗證與協作的虛擬化系統架構  

* Test Engineering:  
  測試工程：

  * A Test framework for acceptance testing.  
    驗收測試用的測試框架  
  * Acceptance test cases and corresponding automated test scripts.  
    驗收測試案例與對應的自動化測試腳本  

## Acceptance Criteria ｜ 驗收標準

* Software Engineering:  
  軟體工程：

  * Requirements are clearly documented, unambiguous, and reviewable.  
    需求文件清楚、無歧義且可審查
  * The system framework is available in a virtual environment.  
    系統框架可於虛擬環境中運行

* Test Engineering:  
  測試工程：

  * A Test framework is defined for acceptance test automation.  
    定義用於驗收測試自動化的測試框架
  * Acceptance test cases and automated scripts are implemented and traceable to requirements.  
    驗收測試案例與自動化腳本已實作，且可追溯到需求
  * Test automation supports both script generation and execution.  
    測試自動化支援腳本生成與執行
  * At least 80% of error-handling scenarios are covered by test cases.  
    測試案例需覆蓋至少 80% 的錯誤處理情境

  * Test Case Pool:  
    測試案例池：

    * Each test step is maintained as a reusable unit in the Test Case Pool.  
        每個測試步驟皆以可重用單元形式維護於測試案例池中
    * The BDD workflow assembles test scripts by reusing steps from the Test Case Pool.  
        BDD 流程透過重用測試案例池中的步驟來組合測試腳本

## Milestone ｜ 里程碑

![Business Milestone](./img/BusinessDevMilestone.drawio.svg)  

---

## License｜授權條款

![BY NC ND](../img/Cc-by-nc-sa.png)  
Business Development Phase © 2016 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
商業開發階段 © 2016 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

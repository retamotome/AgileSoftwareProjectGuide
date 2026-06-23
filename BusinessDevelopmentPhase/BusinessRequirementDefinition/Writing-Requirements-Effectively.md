# Writing Requirements Effectively ｜ 撰寫需求的高效方法

## Overview ｜ 概述

The core of the **Agile mindset** is: **identify what truly matters, do the right things, in the right way, and deliver value**.  
**敏捷思維**的核心是：**釐清真正重要的事，做對的事，用對的方法，並交付價值**。  

At this stage, focus on defining requirements at the **business level**—avoid technical details.  
在此階段，應專注於**商業層級的需求定義**，避免涉及技術細節。  

All **Feature Requirements** should include **Specification by Example** scenario tables. These help create clear, testable acceptance criteria align with BDD principle.  
所有 **功能需求（Feature Requirements）** 應包含 **Specification by Example（範例式規格）** 的情境表格，以建立清楚且可測試的驗收標準，並符合 BDD 原則。  

> [!note]     
> BDD (Behavior-Driven Development) is mainly used for acceptance tests.  
> BDD（行為驅動開發）主要用於驗收測試。  


## Prerequisites ｜ 前置知識

For more background, see:  
更多背景知識請參考：

- [Lesson 01: Agile and Software Engineering 敏捷與軟體工程全覽](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-11-09_AgileProjectMgt/README.md)
- [Lesson 02: Business Requirement Definition 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md)  
- [How To Run a Design Thinking Workshop](https://facilitator.com/blog/how-to-run-a-design-thinking-workshop)  

Recommended books:  
推薦書籍：

- [Specification by Example](https://www.amazon.com/Specification-Example-Successful-Deliver-Software/dp/1617290084)  
[Specification by Example 中文版：團隊如何交付正確的軟體](https://www.tenlong.com.tw/products/9789862019481)
- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)  
[領域驅動設計：軟體核心複雜度的解決方法](https://www.books.com.tw/products/0010821330)


## Example ｜ 範例
- [Business Requirement Example ｜ 商業需求範例](./Example/Business-Requirement-Example.md)  
  BR-001: Secure User Login ｜ BR-001：強化使用者登入安全  
- [Feature Requirement Example ｜ 功能需求範例](./Example/Feature-Requirement-Example.md)   
  FR-001: User Authentication ｜ FR-001: 使用者驗證  
    - This one of the features that support BR-001.  
      這是根據 BR-001 進行細分的其中一項功能需求。 
    

## Procedure ｜ 流程

Requirements are created following the approach in [Lesson 02: Business Requirement Definition 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md).  
需求建立需依循 [Lesson 02: Business Requirement Definition 商業需求定義](https://github.com/retamotome/SelfDirectedTraining/blob/main/AgileSoftwareEngineering/2025-12-08_BusinessReqDef/README.md) 的方法。  

Each requirement should include tables which be presented using the method from [Specification by Example](https://www.amazon.com/Specification-Example-Successful-Deliver-Software/dp/1617290084):  
每一項需求應包含以下表格，並採用 [Specification by Example](https://www.tenlong.com.tw/products/9789862019481) 方法呈現：

- **Risk and Solution** ｜ 風險與對策  
- **Normal Case** ｜ 正常情境  
- **Error Handling** ｜ 錯誤處理  

These tables are then used to generate **acceptance** test cases in a **pseudocode-like** format, such as [Gherkin](https://cucumber.io/docs/gherkin/).  
這些表格接著可轉換為類似**偽程式碼（pseudocode）**格式的**驗收測試案例**，例如使用 [Gherkin](https://cucumber.io/docs/gherkin/)。  

Today, these can be run directly as tests using supported frameworks. This approach predates the term *“Vibe Coding”*.  
現今可透過支援框架直接執行，這種方法甚至早於 *Vibe Coding* 概念出現。  

In the **Identification** phase, use **5W1H** (Who, What, When, Where, Why, How) to clarify each requirement table.  
在 **需求辨識** 階段，應使用 **5W1H**（誰、什麼、何時、何地、為何、如何）釐清每個需求表格。  

These are then expressed in pseudocode-like syntax, as shown below, to serve as corresponding **acceptance** test cases for the requirement.  
之後將其轉換為類似偽程式碼語法，作為對應的**驗收測試案例**。  


### Pseudocode-Like Syntax Example (Gherkin) ｜ 偽程式語法範例
```
Feature: Brief description of what is needed
   In order to achieve a business value
   As a system actor
   I want to gain a beneficial outcome

   Scenario: A specific business situation
      Given some precondition
         And another precondition
      When an action occurs
         And another action
      Then a testable outcome is achieved
         And another result can be checked

   Scenario: Another situation
      ...
```
  
These acceptance test cases can be executed:  
這些驗收測試可透過以下方式執行：  

- using supported frameworks  
  使用測試框架  
- integrated into CI/CD pipelines  
  整合至 CI/CD 流程  
- automated with code-then-run AI agents  
  使用 AI 自動生成並執行  

> [!note]    
> While Gherkin is used as an example here, you are not restricted to any specific language or syntax. Use whatever format is clear and effective for your team.  
> 雖然此處以 Gherkin 為例，但不限於特定語法或語言，可依團隊需求選擇最清楚且有效的方式。  
>
> Likewise, when generating code for tests, you may use C++, Python, or any language you prefer.  
> 同樣地，測試程式碼可使用 C++、Python 或任何適合的語言。

---

**Learn More:**
- [如何寫出好的語法展示你的 Specification By Examples](https://ithelp.ithome.com.tw/articles/10226615)
- [Behat Gherkin Execution Example](https://docs.behat.org/en/v2.5/guides/1.gherkin.html)

---

# License｜授權條款

![BY NC ND](../../img/Cc-by-nc-sa.png)  
Writing Requirements Effectively © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
撰寫需求的高效方法 © 2026 作者 潘貞元（Reta Pan），採用  [姓名標示－非商業性－相同方式分享 4.0 國際](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en) 授權。  

---

> [!note]  
> **AI Translation Notice | AI 翻譯說明**  
> The Chinese content of this document has been generated through AI-based translation and is presented using Traditional Chinese characters and terminology commonly used in Taiwan to ensure clarity, localization, and readability.  
> 本文之中文內容係由人工智慧（AI）翻譯產出，並採用正體中文及台灣常用語彙進行表述，以確保內容符合在地語言之使用與閱讀習慣。  

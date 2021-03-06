---
title: "[閱讀筆記] 單元測試的藝術 Part I"
date: 2020-02-23T23:58:24+08:00
tags: [testing,unit testing,閱讀筆記]
categories: [engineering]
---

# 單元測試的藝術 - 入門篇 (Ch1 & Ch2)

## 定義們

### 被測試系統 SUT（System Under Test）

> 被測試程式所測試的對象，可以是函數，可以是類別，可以是一個複雜的元件，也可以是一個軟體。
一般來說

### 工作單元、使用案例 Use Case

> 從呼叫系統的一個**公開方法**，到產生一個測試**可見最終結果**，在期間這個系統所發生的行為統稱為一個工作單元。

**可見的最終結果**可以是

1. 回傳值（如果公開方法有回傳值的話）
2. 系統**可見的**狀態或行為改變，不需要查詢私有狀態就能取得。
3. 呼叫一個不受測試所控制的第三方系統，這個第三方系統不回傳任何值或者回傳值不被系統使用。

### 單元測試 Unit Test

> 一個單元測試是一段自動化的程式碼，這段程式會呼叫被測試的工作單元，之後對這個單元的最終結果的某些假設或期望進行驗證。

**特質**

> 執行起來快速。可靠、易讀、並且很容易維護。只要產品程式碼不發生變化，單元測試的執行結果是穩定一致的。

* 自動化的，且可以被重複執行的。
* 容易被實現。
* 到第二天還有存在意義，非臨時性的。
* 任何人都可以按個按鈕執行它。
* 執行速度很快。
* 執行結果一致。
* 應該要能完全掌控受測單元。
* 完全被隔離的，獨立於其他測試。
* 如果執行結果失敗，能夠簡單清楚的呈現期望為何以及發生問題的原因在哪。

### 整合測試 Integration Test

> 對一個工作單元進行測試，而這個測試對被測試的單元並沒有完全的控制，而是使用該單元**一個或多個真實依賴的相依物件**，例如時間、網路、資料庫、執行緒或亂數產生器等等。

**特徵**：一次測試的東西太多。

### 測試驅動開發 TDD

```
loop do
  撰寫符合期望的測試 - ( 紅燈 ) > 撰寫或修改產品程式碼 -> ( 綠燈 ) > 重構 - ( 綠燈 )
end
```

### 測試框架

框架是把重複做的東西，整理成好用的工具包，讓開發人員可以不用重新造車。而測試框架就是專門解決撰寫測試程式碼中重複的步驟，讓測試程式結構化、輕易執行所有測試以及協助確認結果。

## 原則與建議

### 關於撰寫測試

* 測試程式內部架構：準備 **A**rrange -> 操作 **A**ct -> 驗證 **A**ssert（3A）。
* 避免在 setup 中時做 Arrange 的步驟，3A 的測試程式碼應該靠進一點。（就像寫工作單元程式一樣）。
* 避免使用 Fixture 或者是在 teardown 中清理測試環境，這限制讓我們可以仔細思考，撰寫的測試是不是偏向整合測試，而不是單元測試。從而思考工作單元的設計和內部行為不夠獨立。除非是要重設 Shared Singleton 或某些常數。
* 測試程式的命名規則與一般程式命名規則不同，測試程式的命名重點在表達**受測單元**、**測試情境**以及**期望結果**。
* 善用參數化來簡化重複類似的測試程式。
* 測試程式的驗證目標應該是工作單元的三種可見的最終結果。
* 透過思考正向、反向以及例外三種情境來設計測試程式。

**自我檢測**

* 兩週前所寫的測試還能正常執行並得到結果嗎？
* 兩個月前撰寫的測試，團隊任何一個人都能正常執行它並得到結果嗎？
* 能在幾分鐘內跑完所有的單元測試嗎？
* 能一鍵執行所有寫過的單元測試嗎？
* 能在幾分鐘內寫出一個基本的單元測試嗎？

答案應該都要是**可以**，才代表你在寫單元測試。
（某些部分有點武斷）

### 關於工作單元

* 不需要盡可能小，適當的大小可以讓測試更容易維護。（更傾向於維護商業邏輯的層面，有 BDD 的味道。）
* 小心過度指定（ overspecification ）。

### 關於很好的執行測試驅動開發 TDD

1. 善於撰寫優秀的測試。
2. 在寫產品程式碼前先寫測試（避免過度理解工作單元的實作）。
3. 良好的測試設計（涵蓋所有商業邏輯的情境）。
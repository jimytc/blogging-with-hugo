---
title: "[閱讀筆記] 單元測試的藝術 Part II"
date: 2020-02-23T23:58:40+08:00
tags: [testing,unit testing,閱讀筆記]
categories: [engineering]
---

# 單元測試的藝術 - 核心技術篇 (Ch3 - Ch6)

### 回顧

**工作單元的可見最終結果有三。**

1. 有回傳值。
2. 系統**可見的**狀態或行為改變。
3. 呼叫不受測試控制的第三方系統。

**測試的種類**

1. 單元測試
2. 整合測試
3. (New) 互動測試

> 互動測試－
> 針對物件如何向其他物件發送訊息的測試。用於測試工作單元的最終結果是與其他系統互動的時候，例如，發送 Log。

## 測試的藝術在於

### 在合適的地方加入或使用一個中介層封裝原本的呼叫行為，藉此可以模擬中介層行為進而測試工作單元。


## 測試程式面臨的挑戰

### 受測單元的外部依賴（External Dependency）

> 無法控制的外部依賴會造成“抑制測試”的現象。
>
> 抑制測試（test-inhibiting）—
> 當程式碼依賴於某個外部資源，即使邏輯是完全正確的，但這種依賴仍可能造成測試失敗。
> 
> 例如，當工作單元的行爲是需要讀取系統中的設定檔案或者網路服務來決定。

## 測試時常用到的技術

### Fake 假物件

> 假物件是通用名詞，可能會是 Stub 也可能會是 Mock，依照使用情境會有相對應的行為。
> 
> 如何分辨 Stub 以及 Mock？
> 
> 最大的差異是測試程式**驗證的對象**是工作單元的運作結果（Stub）還是相依被取代的物件（Mock）。
> 也就是說，Stub 不會造成測試失敗，而 Mock 會。
> 
> **Stub**
> 
> Stub <- (互動) -> 工作單元 <- (驗證) - 測試程式
> 
> **Mock**
> 
> 工作單元 <- (互動) -> Mock <- (驗證) - 測試程式
> 
> 
> 注意：如果還不確定在測試中會把中介層怎麼使用時，建議先以 Fake 來命名，因為當命名是 Stub 或 Mock 的時候，閱讀程式的人就會對其有相對應行為的期待。

### Stub 虛設常式

> 在系統中產生一個可控的替代物件，用以取代外部相依物件（或協作者），用於模擬外部相依的回傳值或者行為。
> 
> 多數測試中應該都以 Stub 為主，因為主要是要排除相依。

### Mock 模擬物件

> 系統中的假物件，它可以拿來驗證被測試物件是否如預期般呼叫這個假物件，因此來使得單元測試執行成功或失敗。
> 
> 每個測試裡最多只有一個模擬物件，而且也要小心測試程式是否對 Stub 物件進行驗證，，這會是過度指定（Overspecification）的徵兆，過度指定會造成明明程式邏輯是對的，但測試卻一直失敗。

### Isolation (Mocking) Framework 隔離（模擬）框架

> 利用語言本身的特性，將 Stub / Mock 等技術包裝起來，進而變成動態包裝的型態。簡單說，就是不用人工手刻那些模擬和驗證的部分。
> 
> 但是要小心使用過多是可能造成測試可讀性降低。
> 而且有時候手刻反而更容易達到效果，畢竟框架能運作也是加入了要求和限制，為了更好的**整合**框架的優勢，程式本身就必須做到某些實作，那些實作有時是不必要的。

## 解除依賴的模式

1. 找到無法順利測試的**介面**。
2. 如果工作單元**直接相依**，透過中介層來取代直接相依的關係。
3. 將相依介面的**底層實作**內容替換成可以控制的程式碼。

### 加入中介層往往代表用上了重構的手法

重構是為了在原本的設計中加入**接縫（Seam）**。

> 接縫（Seam）－
> 在程式碼中可以抽換不同功能的地方，這些功能例如：使用 Stub 類別、增加一個建構函式參數、增加一個可以設定的公開屬性、把一個方法改成可以複寫（Override）的虛擬方法，或是把一個 Delegate 拉出來變成一個參數或屬性供類別外部來決定內容。
> 
> 實作主要遵循開放封閉原則 (OCP，Open-Closed Principle）。

### 重構手法們

> 注意：沒有自動測試保護下，貿然開始重構是很危險的。

**A型重構**

> 將具體類別（Concrete Class）抽象成**介面（Intefaces）**或**委派（Delegates）**
>
> 實際手法：
> 
> 擷取介面已變替換底層實作內容。

**B型重構**

> 重構程式碼，以便將委派或介面的**偽實作**注入至目標物件中。
> 
> 依賴反轉（Inversion of Control）
>
> 依賴注入（Dependency Injection）
> 
> 實際手法：
> 
> 1. 在被測試類別中，注入一個 Stub 的實作。
> 2. 在建構函式注入一個假物件。當需要告訴工作單元的使用者，這個相依是必須的時候。
> 3. 從屬性的讀取或設定中注入假物件。當相依物件並非是必要的時候，但測試程式需要特別控制其行為時。
> 4. 在方法被呼叫前注入一個假物件，採用工廠方法。當受測工作單元取得相依的時間點並非在建構函式執行時，而是在工作單元正要開始工作之前才取得時。當要模擬提供給被測試單元的**輸入（Input）**時 。
>
> 重構成依賴注入的行為，要特別注意：
>
> 1. 建構函式注入導致建構函數的參數過多。 => 把多個參數整合成一個新的類別。或者使用依賴反轉容器協助。但作者少用依賴反轉容器，可讀性會比較好。（明確地表明相依於什麼物件）

### 因為測試程式而破壞了原本的封裝？

物件導向設計的封裝原則是為了限制工作單元的使用者（或程式），從而保證他們會正確地被使用。測試也是另一個使用者，所以當撰寫測試是為了來保證交付的商業邏輯是對的時候，產品程式的設計也應該要考慮**可測試性**。

## 測試健壯性

使用隔離框架和好的測試設計應該要提升測試健壯性，可以從下面五點來思考。

1. 遞迴假物件。這表示透過隔離框架，即使工作單元出現了遞迴式的依賴，依舊可以乾淨的解耦。
2. 預設忽略參數。這主要提升在測試程式的可讀性，不會讓過多的資訊混淆維護者。
3. 大範圍偽造（Wide faking）。
4. 假物件的非嚴格行為。以往嚴格的情境是因為用於測試互動協議，而現在在測試時，並不是非得限制在**只能且必須被呼叫的方法**。
5. 非嚴格模擬物件
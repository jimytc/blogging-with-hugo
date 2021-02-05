---
title: "TypeScript 上手二三事"
date: 2021-02-05T16:06:10+08:00
tags: [typescript,javascript,test,strong type]
categories: [engineering,programming language]
---

經歷過 Java, Ruby, JavaScript(咦?)，最近因為職涯的轉換開始上手 TypeScript。
簡單說 TypeScript 就是 JavaScript 的超集，最大的好處是加強了型別系統。
上手的過程使用了 91 (Joey Chen) 極速開發帶領的 Tennis Kata 練習。

1. 明確的主題可以練習，不會漫無目的的從頭開始學。
2. 新的語言有新的工具鍊，必須要有基本使用的能力。
3. 承 2，我希望可以最快速的達到我在 Ruby 環境的**純 coding** 戰力。

目前的感受

1. 用了一個主題上手真的比較精準和快速，簡單說就是**作中學**。
2. 多了編譯的過程貌似會慢一點點（ TypeScript -> JavaScript -> run Test）。
3. 內建型別推論（Type Inference）在多數情形下可以不去註記型別。
4. 承 3，但是對於某些特別用途，例如 dictionary，就必須要記得註記他。
5. 因為 3 跟 4，反而在寫程式的時候需要特別去思考**要不要作型別註記**，覺得會有一點額外耗腦。
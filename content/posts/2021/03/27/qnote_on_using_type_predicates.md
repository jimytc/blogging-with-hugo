---
title: "Typescript Array.map 後的型別推論技巧 - type predicate"
date: 2021-03-27T08:40:22+08:00
tags: [typescript,javascript]
categories: [engineering,programming language]
---

在集合或陣列類型的資料處理中，偶爾會遇到需要使用 `Array.map` 把所需要的資料從物件中轉化成另一個陣列物件。
例如

```typescript

class Foo {
  bar?: number
  
  constructor(bar?: number) {
    this.bar = bar;
  }
}

let foos: Foo[];
fooArray = [
  new Foo(123),
  new Foo()
];

let bars = foos.map(f => f.bar)
```

在 `tsconfig.json` 中開啟 `"strict": true` 或者 `"strictNullChecks": true` 的情形下。
`bars` 會被推論成是 `(number | undefined)[]`。

這是合理的，但是如果在某些元件中 `foos` 已經能夠確保 `bar` 不會是 `undefined`，我們也許會這樣使用。

```typescript
function getFoosWithValidBar() {
  return [new Foo(123), new Foo(654)];
}

let foos = getFoosWithValidBar();
foos.map(t => t.bar).forEach(b => console.log(b * 2));
```

在同樣的 `tsconfig.json` 設定下，TypeScript 的編譯器會跟你通知 `b` 會有問題。

> `TS2532: Object is possibly 'undefined'.`

目前的處理方法是要使用 `filter` 搭配 [Type Predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates) 處理。

```typescript
function getFoosWithValidBar() {
  return [new Foo(123), new Foo(654)];
}

function isNumber(n: number | undefined): n is number {
  return typeof(n) === 'number';
}

let foos = getFoosWithValidBar();
foos.map(t => t.bar).filter(isNumber).forEach(b => console.log(b * 2));
```

基本上使用 `strict` 以後，在寫程式上就會需要很多這種類似的型別檢查，好處是減少 runtime error，但相對這對一開始的模組或元件設計就要很仔細了。

老實說我還是比較喜歡 duck typing 一點，減少一些這些特殊的 runtime 檢查，但是用單元測試去覆蓋，確保元件的輸出都符合需求。

話說 型別註記、型別推論和測試 都是確保程式執行不會遇到奇怪的狀態，但是寫測試還可以多保護商業邏輯，我覺得投資報酬相對高啊。

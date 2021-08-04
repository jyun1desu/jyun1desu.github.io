---
title: 可測試的 JavaScript Ch2 複雜度（上）
date: 2021-08-04 22:09:54
cover: img/covers/js.png
categories:
- 《可測試的 JavaScript》 讀書筆記
tags:
- 可測試的 JavaScript
- 讀書筆記
- 複雜度
---
雖然完全精確測量程式複雜度的方法是不存在的，但有幾個重要的指標能夠測量出部分的程式複雜度：
- 程式碼大小(code size)
- 循環複雜度(cyclomatic complexity)
- 扇入
- 扇出

⋯⋯等等可以測量出部分的程式複雜度。使用這些項目去檢視自己的程式碼，是建立可測試的程式的良好開始。
<!-- more -->
## 程式碼大小（Code Size）
雖然整個專案中必要的程式碼的數量不太可能改變，但每個函式的大小、或是每個檔案中的函式數量可以改變的（檔案內的函式應該盡量相關）！

### 命令與查詢

<b>將命令(command)與查詢(query)分開</b>是維持函式最小尺寸的方式之一：
- 命令：可以視為 setters ， 為 做某件事 的函式
- 查詢：可以視為 getters ， 為 傳回某個結果 的函式

使用這種把讀與寫分開的方式撰寫函式，除了能精簡程式、提高可讀性外，測試時的彈性也更高。

不好的示範：
```js=
function configure(values) {
    const fs = require('fs');
    const config = { docRoot: '/root' };
    let key;
    let state;
    
    for(key in values) {
        config[key] = values[key]
    }
    
    try {
        state = fs.statSync(config.docRoot);
        if(!state.isDirectory()){
            throw new Error('is not valid');
        }
    } catch(e) {
        console.log(`** ${config.docRoot} does not exist`)
        return;
    }
    
    //..validate other values...
    return config;
}
```
這段程式中除了將 `values` 中的值寫入 `config` ，還要驗證資料，除了測試寫起來會很龐大、沒有焦點外，也<b>不能單獨驗證`config`的內容</b>。

如果將這段程式碼拆寫：
```js=
function configure(values) {
    const config = { docRoot: '/root' };
    let key;
    
    for(key in values) {
        config[key] = values[key]
    }
    
    validateDocRoot(config);
    validateSomething(config);
    
    return config;
}

function validateDocRoot(config) {
    const fs = require('fs');
    cosnt state = fs.statSync(config.docRoot);
    if(!state.isDirectory(）){
        throw new Error('is not valid');
    }
}

//validate other values...
function validateSomething(config) {
    ...
}
```
瞬間變得清楚多了！也能針對單一項目去做驗證、測試。作者後續還有做一些改良，但這個步驟就很能說明將命令與查詢分開的好處了。

## JSLint 
這本書是 2014 年出版的，現在比較常用的類似工具是 ESLint。用來分析程式碼中是否有**不良的風格 (style)、語法 (syntax) 以及語意 (semantics)**，以提高程式碼的可讀性。

## 循環複雜度 Cyclomatic Complexity
指的是在程式中**無相依關聯**的程式碼數量。
即為了測試所有程式碼，所需撰寫的單元測試之最小數量（測試覆蓋率100%時，所需要最小的單元測試數量）。

>💡 測量工具：
>- JSmeter
>- jecheckstyle + Jenkins
>- VScode 中可以安裝 CodeMetrics

以下是一個循環複雜度 2 的函式：
```js=
const sum = (a, b) => {
    if(typeof a !== typeof b){
        throw new Error("can't sum different types!");
    } else {
        return a + b
    } 
}
```

### 查找表
循環複雜度較高時，通常表示程式碼內有許多 `if/else` 或是 `switch` 陳述句，將之改寫為**查找表**：

```js=
// if/else陳述句
const doSomething = a => {
    if(a === 'x') {
        doX();
    } else if (a === 'y') {
        doY()
    } else {
        doZ();
    }
}

// 查找表
const doSomething = a => {
    const lookup = {
        x: doX,
        y: doY
    };
    const default = doZ;
    
    lookup[a] ? lookup[a]() : default();
}
```
改寫後雖然**沒有**減少需撰寫的單元測試數量，但撰寫的做法有所改變：
- if/else：對單一函式撰寫多個單元測試
```js=
describe('Test doSomething', () => {
    test('a is x', () => {
        ...
    })

    test('a is y', () => {
        ...
    })

    test('default', () => {
        ...
    })
});
```
- 查找表：對多個較小的函式寫對應的單元測試
```js=
describe('Test lookup x', () => {
    test('doX', () => {
        ...
    })
});

describe('Test lookup y', () => {
    test('doY', () => {
        ...
    })
});

describe('Test default', () => {
    test('doZ', () => {
        ...
    })
});
```
可維護性提高，感覺也更好讀了！

## 重複使用 Reuse
減少程式碼的大小最好的方式就是降低撰寫數量。
一個典型的應用程式大約有 85% 以上是使用他人撰寫好的程式碼（套件、框架等等），只有 15% 會是開發者自行撰寫的。

除了善用第三方資源，更重要的是在這 15% 中**重複自己的程式碼**。對於重複出現的程式碼不要有「就多出這一次而已」的心態。 


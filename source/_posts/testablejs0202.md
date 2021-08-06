---
title: 可測試的 JavaScript Ch2 複雜度（下）
date: 2021-08-06 22:46:00
cover: img/covers/js.png
categories:
- 《可測試的 JavaScript》 讀書筆記
tags:
- 可測試的 JavaScript
- 讀書筆記
- 複雜度
- 相依性注入
- 耦合
---
## 扇出與扇入
### 扇出 Fan-out
扇出的定義是一個函式的**內部流數目**加上**該函式更動到的全域資料數目**
以下狀況都可以被算作是函式A的內部流：
- A 呼叫 B
- B 呼叫 A ，且 A 回傳一個值、B 會利用這個值
- C 呼叫 A 和 B ，並把 A 的輸出值傳給 B
<!-- more -->
### 扇入 Fan-in
扇入的定義是**內部流進入該函式的數目**，加上**從該函式取用內容的資料結構數目**。
大多情況下大的扇入數是好的，表示有越多共用程式的重複使用。

### 小結
高的扇入數與扇出數可能暗示這個函示**做了太多事**，需要被重構。


## 耦合 Coupling
扇出是計算一個模組或函式需求的相依模組與物件的數量，耦合則是表達這些模組、物件之間的關聯情況。耦合的程度由高到低分成六個等級：
- 內容耦合(5分)：直接修改物件本身的狀態
```js=
person.name = 'Lotso';
```
- 共用耦合(4分)：兩個物件共用一個全域變數，彼此就是共用耦合關係。
```js=
let global = 'global';

const a = () => {
    global = 'a';
};

const b = () => {
    global = 'b';
}
```
- 控制耦合(3分)：利用旗標(flag)或參數設定控制外部物件的運作：
```js=
const whiteRabbit = new AbstractRabbit({color: 'white'});
```
- 戳記耦合(2分)：經由參數傳遞一筆資料給物件A，A只會用到該資料的**部分內容**
這裡的 `makeBread` 只用到 `bread` 的三個屬性，沒有用到 `name`：
```js=
Oven.prototype.makeBread = function({type, size, flavor}) {
    return new Bread(type, size, flavor)
};

const bread = {
    name: 'bread1', 
    type: 'wheat', 
    size: 99, 
    flavor: 'macha'
};

Oven.makeBread(bread);
```
- 資料耦合(1分)：經由參數傳遞一筆資料給物件A，且沒有將資料的控制權轉給外部物件。在第三章會有更細的探討。
- 無耦合：兩個物件無關～～讚！

將耦合程度越小（資料耦合、無耦合），測試起來越容易！

### 實體 Instantiation
實體化**非獨生(nonsingleton)的全域物件**雖不屬於上述的正規耦合類型，但是一種介於內容耦合與共用耦合之間、**非常緊密**的耦合類型。
- 在不使用此物件時，必須摧毀它。
- 物件實體越少，程式碼複雜度越低。如果發現程式碼用了太多物件、就是時候重新思考程式架構了！

>獨生/非獨生的概念可以參考[這個](https://gist.github.com/eymorale/ba1f8887160cbd749ad3eff6bb28c4a5)，滿好懂的！

### 相依性注入 Dependency Injection
先來看一個緊密耦合的例子：
```js=
function setParty() {
    const djPlayer = new DJPlayer();
    const dishes = new Dishes();
    
    this.placeDJPlayer(djPlayer);
    this.placeDishes(dishes);
}
```
當準備要測試這個程式的時候，需要準備 DJPlayer 物件跟 Dishes 物件（這會使測試變得困難）。單元測試會希望不要依靠外部相依性就能獨立測試 `setParty`。

這時可以使用注入(injection)去減少耦合程度：
```js=
function setParty(djPlayer, dishes) {
    this.placeDJPlayer(djPlayer);
    this.placeDishes(dishes);
}
```

這時候會有個疑問，如果這裡注入了一個物件，那還是有一個地方需要去建立它啊！！！沒錯，作者指出這件事通常會是在**程式或測試的開頭**。這樣的做法使得任意物件更容易與另一個物件做交替。

而且在寫測試時，也可以使用**虛構的版本**來替換注入的物件，使測試更加容易撰寫。

## 本章重點回顧
- 盡可能測量程式碼的複雜度，進而整理程式碼、提高可測試性、可維護性。
- 程式間的相依性（耦合）使得測試變得困難，可以使用注入的方式解決。
- 降低程式碼的複雜度無非就是**了解何處複雜**並且重構。作者推薦兩本書
    - Refactoring: Improving the Design of Existing Code
    - Code Complete: A Practical Handbook of Software Construction
- 完全排除複雜度是不可能的，但可以透過正確的註解、良好的測試使之最小化。
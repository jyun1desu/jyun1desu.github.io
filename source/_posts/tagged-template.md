---
title: 'styled-components 背後的魔法: Tagged template'
date: 2021-07-19 23:11:37
cover: img/covers/js.png
categories:
- JavaScript
tags:
- styled-components
- ES6
- Tagged template
---
寫了一陣子的 styled-components，最近才知道原來`styled.tag`後面的``，就是 ES6 常常在寫的那個（打爆自己的笨頭），它的名字是 backtick。除此之外還發現了神奇的用法，研究了一下覺得很有趣，尤其[這篇文章](https://mxstbr.blog/2016/11/styled-components-magic-explained/)寫得很詳細（官方推薦）！以下是看完之後做的紀錄，如果習慣看英文的話很推薦看看原文～

先從最簡單的開始8！
<!-- more -->
## 範本字面值 Template Literal / Interpolated String Literal

以前要用很多 `+` 串起來的東東，現在只要用 backticks 加上 `${ 變數名稱 }`，整個很清爽～backticks 之間的斷行、空格等也都會被保留！

```jsx=
const myFavorite = 'Lotso';
const flavour = 'strawberries'

// before ES6
const introduceMyBearInES5 = 
	'My favorite bear is '+ myFavorite + ', and his tommy smells like'+ flavour +'.';
// ES6
const introduceMyBearInES6 = 
	`My favorite bear is ${myFavorite}, and his tommy smells like ${flavour}.`;
```

`${ 變數名稱 }` 中除了字串，也能放入各種運算式，甚至是呼叫函式：

```jsx=
const getFullName = ({firstName, lastName}) => {
	return `${firstName} + ${lastName}`
}

const President = {
	firstName: 'Lotso',
	LastName: 'Bear'
	height: 50,
}

const introducePresident = 
	`Our President is ${getFullName(President)}.
	 He is ${President.height > 80 ? 'tall' : 'short'}.
	`
// 'Our President is Lotso Bear. 
//  He is short.'
```

## 標籤模版 tagged template

標籤模板指的是在函式後方直接用上 backticks ，很有趣的是印出來的argument會是陣列！

```jsx=
const printSomething = (...args) => {
	console.log(...args)
}

//一般的函式
printSomething('a', 'b') // 'a b'

//標籤模版
printSomething`` // ['']
printSomething`I love Lotso` // ['I love Lotso']
```

雖然有點差異，但是其實跟一般的函式相差不遠。但如果模板字符串中有使用到 placeholder (就是那個熟悉的`${...}`)，就會發生很神奇的事情！

```jsx=
const myFavorite = 'Lotso';
const printSomething = (...args) => {
	console.log(...args)
}

/* 呼叫 printSomething, 並將模版字符串作為引數代入 */
printSomething(`My favorite bear is ${myFavorite}.`) 
// 'My favorite bear is Lotso.'

/* call printSomething as a tagged template literal(標籤樣板字面值)*/
printSomething`My favorite bear is ${myFavorite}.`
//["My favorite bear is ", "."], "Lotso"
```

當非使用一般代入的形式、而是在 `printSomething` 後方使用標籤樣板字面值時，會得到一 `Array` 作為第一個變數，第二個則是placeholder `myFavorite` 的值。而第一個 `Array` 內的值看起來是由 `myFavorite` 所在的位置去區隔的

再來多塞一個插入值看看：

```jsx=
const myFavorite = 'Lotso';
const flavour = 'strawberries'

printSomething`My favorite bear is ${myFavorite}, and his tommy smells like ${flavour}.`;
/**
* ["My favorite bear is ", ", and his tommy smells like ", "."] 
* "Lotso" 
* "strawberries"
**/
```

在回傳值中，第一個變數會是一陣列，內有非插入值的所有字串；而插入值則會依序排列在後方的變數中。再來看一次比較：

```jsx=
const myFavorite = 'Lotso';
const flavour = 'strawberries'

printSomething(`My favorite bear is ${myFavorite}, and his tommy smells like ${flavour}.`) 
// "My favorite bear is Lotso, and his tommy smells like strawberries."

printSomething`My favorite bear is ${myFavorite}, and his tommy smells like ${flavour}.`;
/**
* ["My favorite bear is ", ", and his tommy smells like ", "."] 
* "Lotso" 
* "strawberries"
**/
```

特點介紹完畢！那它神奇好用的地方在哪裡呢！！！請看以下：

### 在 styled-components 中的妙用

簡單舉一個 styled-components 中的語法：

```jsx=
const Button = styled.button`
  font-size: ${props => props.primary ? '2em' : '1em'};
`

// font-size: 2em;
<Button primary />

// font-size: 1em;
<Button />
```

寫起來很順，從來沒想過這是怎麼做到的XD
tagged template 起了什麼作用？

先來看放函式在插入值中會印出什麼：

```jsx=
const printSomething = (...args) => {
	console.log(args)
}

printSomething(`font-size: ${(primary) => primary ? '2em' : '1em'}`)
// "font-size: (primary) => primary ? '2em' : '1em'"  <- 只是一坨字串

printSomething`font-size: ${(primary) => primary ? '2em' : '1em'}`
/**
* ["font-size: ", "",]
* (primary) => primary ? '2em' : '1em'  <- 好像是字串又好像是函式
**/
```

magic～～～這裡的 `(primary) => primary ? '2em' : '1em'` 是貨真價實的函式！

![](https://imgur.com/5r5SMbq.jpg)

既然是函式，那就只剩下執行它這個任務了！
來改寫一下很簡陋的 `printSomething`，順便改名成 `printStyles` ：

```jsx=
let protoStyledComponent = {
	printStyles(...args) {
		const [literalArr, ...placeholders] = args;
	
		const styles = placeholders.reduce((result, expr, index) => {
			/* 最大重點：如果是函式就執行它！ */
			const value = typeof expr === 'function' ? expr(this.props) : expr;
			return result + value + literalArr[index + 1];
		}, literalArr[0]);
	
		return styles;
	}
}

let component = Object.create(protoStyledComponent);
component.props = { primary: true, color: 'pink' }; // 塞一些假的props

component.printStyles`
font-size: ${(props) => props.primary ? '2em' : '1em'};
color: ${(props)=> props.color}
`

/*
"font-size: 2em;
color: pink;"
*/
```

登登！！！好神奇R～～～突然多瞭解了一點點每天在寫的東西，寫起來好像也更頭腦清楚了！後續的更多底層處理，可以看[這篇](https://www.gushiciku.cn/pl/g2cN/zh-tw)。

## 參考資料
[The magic behind 💅 styled-components](https://mxstbr.blog/2016/11/styled-components-magic-explained)


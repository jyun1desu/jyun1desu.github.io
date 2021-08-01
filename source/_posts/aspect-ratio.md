---
title: aspect-ratio：等比例縮放區塊的屬性來了！
date: 2021-08-01 16:20:20
cover: img/covers/css.png
categories:
- CSS
tags:
- css
---
開發的時候常常遇到需要固定區塊比例的情境，而且通常長寬都不是絕對的數值。之前都是利用 `padding` 的特性去做等比例區塊。但現在有了更方便的 `aspect-ratio`，加上 `vw`/`vh` ，整理了 3 個可以使用的方法：
<!-- more -->
## 1. vw or vh 
`vw`、`vh`分別指瀏覽器的寬度以及高度，這個方法在佈局比較簡單的行動版裝置上比較可行。
以下是一個長寬都是50%螢幕寬度的正方形：
```css=
.square-box {
    width: 50vw;
    height: 50vw;
}
```
## 2. padding + position 
padding 有個特性——給予的值是百分比時，百分比的數值是基於<b>父層元素的<font color="#198864">寬度</font></b>、而不是元素本身。
假設現在需要一個 16:9 的區塊，可以這樣做：
透過 
`width: 100%` ：取得 `.parent` 寬度的 100%;
`padding-bottom: 56.25%` ：取得 `.parent` 寬度的 56.25%;
製作出 16:9 等比例縮放的外層 `.outer` 後，再絕對定位內層 `.content` ，並將寬高皆設置成 100%，就能得到一個與外層相同大小，但可以任意佈局內部元素的區塊了！
<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="oNWyYjP" data-user="jyun1desu" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/jyun1desu/pen/oNWyYjP">
  aspect-radio</a> by jyun1 (<a href="https://codepen.io/jyun1desu">@jyun1desu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

---
懶得計算的話，也可以用 `calc()`：
```css=
.child {
    width: 100%; 
    padding-bottom: calc(9/16 * 100%);
}
```
## 3. 簡單暴力的 aspect-ratio
`aspect-ratio` 是滿新的 CSS 屬性，支援度可以看[這裡](https://caniuse.com/?search=aspect-ratio)。只要使用 `aspect-ratio: ratio;`， ratio 的表現方式是`width/height`（有夠簡單！）。
也能設置最大寬度、高度，依然會維持等比例，感覺這點是方法2做不到的。
<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="bGWKwmX" data-user="jyun1desu" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/jyun1desu/pen/bGWKwmX">
  aspect-radio</a> by jyun1 (<a href="https://codepen.io/jyun1desu">@jyun1desu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

### 參考資料
[W3C Aspect Ratios](https://www.w3.org/TR/css-sizing-4/#ratios)
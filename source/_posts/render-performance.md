---
title: transform vs position：瀏覽器渲染的簡單介紹
date: 2021-08-14 01:38:56
cover: img/covers/web.png
categories:
- Web
tags:
- web
- rendering performance
- 效能調校
---
有天睡覺前突然想到（就是這麼突然），用 transform 跟 left, top 都可以改變元素位置的話，那是差在哪🧐？就查了一下，原來跟瀏覽器的渲染有關係啊。
<!-- more -->
### 重要的數字：60fps
作為前端工程師最重要的任務之一應該就是讓你的網頁互動多多、又滑又順，使用者、PM大家都開開心心。任何卡卡或是顫抖的感覺（Juddering）都會破壞開開心心。

#### 為什麼會卡卡？
現今大多瀏覽器的更新頻率都是**每秒60幀**，換算下來每個畫面（1幀）大約有 16ms 左右來渲染畫面，聽起來已經有夠少。但瀏覽器它還有其他事要處理，所以大約只能撥 **10ms** 來處理畫面。

當事情太多，沒辦法在時間內處理完時，瀏覽器的畫面率（每秒幀數）就會下降，使用者就會感覺到卡卡的。

而提升效能——讓瀏覽器減少要做的事、**只做必要的事**，就變得很重要了！

-----
## 像素管道(The pixel pipeline)
瀏覽器渲染的過程有五個主要步驟，也是前端工程師最能控制的部分：
![](https://i.imgur.com/RX5fLaY.jpg)
圖片來源：[Rendering Performance](https://developers.google.com/web/fundamentals/performance/rendering)

- JavaScript：主要用來處理**形成視覺變更的工作**，
- Style：套用 CSS 規則在對應的元素上，並計算元素的最終樣式
- Layout：計算**元素間**的版面配置。一個元素往往可以影響許多其他元素。
- Paint：繪製元素的視覺化效果，如文字、圖片、顏色、陰影、邊框⋯⋯等。描繪通常是在多重的層(layers)上進行。
- Composite：合成步驟。完成描繪後，需要將 layers 以正確的順序描繪至螢幕（尤其是有重疊的狀況發生時）。

當 Layout、Paint、Compositing 任一步驟被觸發時，它後面的步驟也會跟著被觸發。無需處理到的步驟則會被跳過。有以下這三種情境：

### change Layout property ： 
![](https://i.imgur.com/ArJvEbx.jpg)
更新與版面配置有關的屬性，如：
- width、height
- left、right、top、bottom
- margin

由於這些屬性會影響元素與其他元素間的關係，瀏覽器不只需要處理該元素、也必須檢查所有其他元素。之後才重新將整個網頁排版(reflow)，並進行後續的渲染行為。

### change Paint property
![](https://i.imgur.com/3l58vQG.jpg)
更新與元素視覺相關、而無改面版面配置的屬性，像是：
- background-image
- color 系列
- text 系列
- box-shadows
- border

### change Composite property
![](https://i.imgur.com/wyq8jha.jpg)
若更改的屬性非 Layout 、 Paint 屬性，就只會觸發 Composite。
目前來說是指這兩個屬性：
- transform
- opacity

> 💡 好用的查詢工具 💡
> [CSS Triggers](https://csstriggers.com/) 可以查詢各 css 屬性會觸發的階段

看到這裡有燈泡冒出來嗎 💡💡💡💡💡？
使用 `left: 10px` 與 `transform: translateX(10px)` 時看起來的效果可能一樣（也可能 transform 更 smoothy 一點），但瀏覽器渲染所需的成本差異卻很大。哇喔真的好神奇！


另外 Style 也是有可以優化的地方，像是選擇器的複雜度就會影響瀏覽器查找的時間。希望之後也可以來寫一篇！推薦可以閱讀：
- [Optimizing CSS: ID Selectors and Other Myths](https://www.sitepoint.com/optimizing-css-id-selectors-and-other-myths/)

---
### 參考資料(兼推薦閱讀)
這兩篇是同個 Google 大師寫的，簡短有力：
- [Simplify Paint Complexity and Reduce Paint Areas](https://developers.google.com/web/fundamentals/performance/rendering/simplify-paint-complexity-and-reduce-paint-areas)
- [Rendering Performance](https://developers.google.com/web/fundamentals/performance/rendering)


其他

[Stick to Compositor-Only Properties and Manage Layer Count](https://developers.google.com/web/fundamentals/performance/rendering/stick-to-compositor-only-properties-and-manage-layer-count)
[Smooth as Butter: Achieving 60 FPS Animations with CSS3](https://medium.com/outsystems-experts/how-to-achieve-60-fps-animations-with-css3-db7b98610108)
[CSS GPU Animation: Doing It Right](https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/)
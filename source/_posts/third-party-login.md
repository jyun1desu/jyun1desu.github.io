---
title: 手動建立 Google 、 Facebook 第三方登入
date: 2021-07-19 20:44:05
cover: img/covers/js.png
categories:
- JavaScript
tags:
- 第三方登入
- google登入
- facebook登入
---
前陣子遇到客戶反應 Facebook、Google 第三方登入 API 在
- instagram 、 Line 等內建瀏覽器(in-app browser)
- 無痕瀏覽模式
  
無法使用的問題，花了一段時間糾結跟解決QQ。意外學了很多以前不知道的東西，記錄一下！
<!-- more -->
## 原先的做法
利用 Facebook、Google 提供的 API ，使用起來很單純：開啟新分頁，使用者完成登入流程後回傳資料給原先的頁面並自動關閉。

### 🤯  in-app browser 的問題： 開啟新分頁
這類瀏覽器大多 **沒有分頁功能，會直接在同一頁做切換**。原先等待回傳的那頁就消失了，也就無法收到回傳。

### 🤯 無痕瀏覽的問題： cookie 被阻擋
Safari 和 Chrome 的表現不太一樣。Safari （14.0.3）沒問題，但 [Chrome 會擋 cookie 進而影響到自家的登入API](https://developers.google.com/identity/sign-in/web/troubleshooting#third-party_cookies_and_data_blocked) (傻眼)

![](https://i.imgur.com/n2duZGn.jpg)
chrome 無痕模式下，使用 `gapi.auth2.init` 方法會出現的error message。

## 手動建立登入流程：one-page-flow
主要是在同一頁面利用轉址的方式進行登入，並在最後將登入資料帶在轉回網址的參數中。從
- 原先：按下登入 → 開啟新分頁 → 登入 → 關閉分頁 → 原先登入頁接收 API 回傳
- 改成：按下登入 → 前往登入頁 → 登入 → 導回指定頁面，資料帶在網址中 

只要設定好 **登入頁的網址** 就差不多成功了；導回後，也只要在 route 中去增加一些對參數的判斷、處理就好了。

### Facebook
覺得[臉書寫的文件](https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow)比較好懂，先拿臉書來當範例

```js
https://www.facebook.com/v10.0/dialog/oauth?
    client_id={app-id} //required
    &redirect_uri={登入完後的回傳址} //required
    &state={防止跨網站偽造的的字串，也會被放在回傳網址中} //required
    &response_type={回傳資料類型} //optional，預設為 code
    &scope={你要向使用者要求的帳號權限} //optional
```

### Google
除了可以看 [JS Web Apps](https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow#oauth-2.0-endpoints) ，也可以一起看 [Mobile and Desktop Apps](https://developers.google.com/identity/protocols/oauth2/native-app) 裡的內容，寫起來跟 facebook 幾乎完全一樣 XD：

```js
https://accounts.google.com/o/oauth2/v2/auth?
    client_id={app-id}
    &redirect_uri=https://jyun1desu.com/login
    &state=google
    &response_type=code
    &scope=email%20profile
```

### 比較需要注意的參數：
- **response_type**
    - `code`： 跟 API 直接取得 `access_token` 不同，需要走 server-side login：前端給後端 `code` ，由後端去換取 `access_token`。
    - `token`： 直接取得 `access_token`。

    雖然直接設定 `token` 很方便，但資料不會以參數(?)、而是以網址片段(#)的方式存在。覺得這樣取資料有點麻煩、也有工程師建議 server-side login 後端的權限管理會比較踏實，就決定取 `code` 了（後端人很好）。

- **redirect_uri**
    要記得到當初在 facebook developer / google cloud 建立的應用程式中， **有效的 OAuth 重新導向 URI** 新增有效網址。不在列表中的網址都會被擋下來(路徑、參數也都要填上)。
 
## 可是我想開啟新頁面 R

撇除在 in-app browser 沒有分頁功能先不理它XD，這個方法最大的缺點是頁面都會在同一頁跳轉，跟多數使用者習慣的體驗是不一樣的。一開始想法是：

- in-app browser → 手動流程(不開新分頁)
- 一般瀏覽器 
    - 正常模式 → 保留使用API
    - 無痕模式 → 手動流程(不開新分頁)

結果找不到判斷無痕的可靠方法...🥲 ，只能改成這樣：
- in-app browser → 手動流程(不開新分頁)
- 一般瀏覽器 → 手動流程(不開新分頁⋯⋯但好想開R！！！)

於是開始想要怎麼模擬 API **開啟新分頁**、**傳資料給隔壁分頁**、**自動關閉** 的行為：

- 開啟新分頁：用 `window.open()` 即可 XD 
- 自動關閉：這個也很簡單，用 `window.close()` 即可 XD
- 傳資料給隔壁分頁：這個好難RRRRRRR！！！！！

糾結好幾天後（連 `localStorage` 搭配 `visibilitychangeEvent` 都試了🥲），發現很讚的

## window.postMessage()

[MDN的解釋很詳細](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) ，以前居然不知道這個 API，本次最大收穫～～～

`postMessage` 可以實現同個瀏覽器下的溝通：tab間、彈出視窗與主視窗間、甚至和嵌入的 `iframe` 也可以～太方便啦！使用起來也滿直觀的：

`postMessage(message, targetOrigin, transfer)` 
- message：要傳送的資料，必填。
- targetOrigin：限制接收對象的 origin ，使用字串 * 則代表無限制，必填。
- transfer：看無....但好像不太用到。

監聽來自他方的 `message` 也很簡單：
`window.addEventListener('message', callback)`

## 簡易的範例
在按下第三方登入鈕的時候判斷是否是 in-app browser
- true： 不開啟新頁面
- false： 開啟新頁面、 redirect_uri 多設定一個參數，判斷回來要做一些事
---
點擊第三方登入鈕觸發 `openThirdPartyAuthPage`：

```jsx
const addReciveMessageListener = () => {
    const reciveEvent = e => {
        const data = e.data
        if(origin === window.location.origin && data.code) {
            doSomethingToLogin(data.code)
            window.removeEventListener('message', reciveEvent, false);
        }
    };
    window.addEventListener('message', reciveEvent, false);
};

const getOAuthPageURL = (platform, isWebview) => {
	const redirect = isWebview? '.../login' : '.../login?type=auto-close';
	switch(platform) {
            case 'facebook':
                return `https:// .....&redirect_uri=${redirectUri}....`;
            case 'google':
                return `https:// .....&redirect_uri=${redirectUri}....`;
        }
}

const toThirdPartyOAuthPage = platform => {
	const isWebview = ...; //做一些 in-app browser 的判斷
	const authPageURL = getOAuthPageURL(platform, isWebview);

	if(isWebview){
            window.location.href = authPageURL;
	} else {
            addReciveMessageListener();
            window.open(authPageURL);
	}
}
```

視窗回到 `redirect_uri` 後，在 route 中處理登入行為：
```jsx=
// routes/Login/index.js

const { code, type, ... } = 網址的query;
const sendDataToLoginPage = () => {
    const loginData = { code };
    window.opener.postMessage(loginData, 'http://jyun1desu.com');
}

if(code) {
    if(type === 'auto-close') {
            // normal browser redirect page
            sendDataToLoginPage();
            window.close();
    } else{
            // in-app browser redirect page
            doSomethingToLogin(code)
    }
}
```

成功的時候好感動啊～～～ 🎉🎉🎉

找資料的過程中也有看到 [Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API) ，雖然支援度還很悲劇，感覺是能先看起來的東西。最後謝謝 hahow ， 一度覺得解決不了的無很多問題，去看了 hahow 的網站之後才發現：嗯！果然是自己太淺ㄌ><，就是有人做得到啊！！！而且寫這篇文章的時候，突然覺得是不是其實滿簡單的⋯⋯（但第一次碰到就是覺得很難QQ）。

以上！希望能幫助到正在困擾同樣問題的人 🎉


## 參考資料
[Google OAuth](https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow)
[Facebook OAuth](https://developers.facebook.com/docs/facebook-login/web?locale=zh_TW)
[postMessage API](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)

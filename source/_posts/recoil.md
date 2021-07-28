---
title: 'Recoil: React官方推出的狀態管理工具'
date: 2021-07-28 23:47:21
cover: img/covers/1.png
categories:
- JavaScript
tags:
- Recoil
- React
---
Recoil 是 Facebook 在2020推出的狀態管理 Library，還在實驗階段（不推薦用在大型專案）。雖然不像 redux 有各種神奇的用法，但比起來少了很多固定的語法要寫，使用起來很簡單！

## 核心概念
一圖以蔽之的概念圖：
![recoil概念圖](https://miro.medium.com/max/2000/1*LEROK_E7fJWr4chqlN93Dg.jpeg)
<!-- more -->
Recoil中有兩種 `shared state` ，都能夠被訂閱、更新：
- **atom**：
    是 `shared state` 的基本單位。
- **selector**：
    為一 pure function ，透過輸入其他的 `shared state` ，輸出成需要的資料(derived data)。在輸入的 `shared state` 更新時才會重新計算。

在`shared state`更新時，只有訂閱該 `shared state` 的元件會重新渲染。

## atom()
每個`atom`都需要 **unique key** 和**預設值**：
```jsx=
const todoList = atom({
    key: 'todoList',
    default: [{task: '看一本書',isComplete: false},
              {task: '打掃家裡',isComplete: true}]
});

const todoListfilter = atom({
    key: 'todoListfilter',
    default: 'default'
});
```
在`component`中，Recoil 提供了幾個 API ：
跟`useState()`很像，但括號內放入的不是預設值，是`atom`。
- `useRecoilValue()`：取值
```jsx =
const list = useRecoilValue(todoList);
```
- `useSetRecoilState()`：設值
使用這個 hook 時，`shared state`改變時component不會重新渲染。
```jsx =
const setTodos = useSetRecoilState(todoList);
```
- `useRecoilState()`：取值及設值
```jsx =
const [list,setTodos] = useRecoilState(todoList) 
```
- `useResetRecoilState()`：能將`atom`設定為預設值，


## selector()
每個`selector`也都需要 **unique key**，以及`get`、`set`(非必要) method。

以 todoList 為例，有了 **todoList** 和目前的 **filter** 狀態，就可以利用`selector`計算當前頁面要顯示的資料了。
```jsx=
const filteredTodos = selector({
    key: 'filteredTodos',
    
    get: ({get}) => {
    const filter = get(todoListfilter);
    const list = get(todoList);

    switch (filter) {
      case 'done':
        return list.filter((item) => item.isComplete);
      case 'undone':
        return list.filter((item) => !item.isComplete);
      default:
        return list;
    }
  },
  
  //optional
    set: ({get, set}, newValue) => { ... }
})

const TodoList = () => {
    const todoList = useRecoilValue(filteredTodos);
    return 
    (<ul>
        {todoList.map(todo=>(<li>{todo.task}</li>))}
    </ul>)
}

```
取值和設值都可以透過`get`取得其他`shared state`的資料。

## 非同步資料串接
`atom`及`selector`都可以利用**非同步**的方式串接資料：
```jsx=
const userID = atom({
    key:  'userID',
    default: null,
});

const userData = selector({
    key: 'userData',
    get: ({get}) => {
        const id = get(userID)
        const response = await fetchUserData({id});
        return response.data;
    }
})
```
利用`useRecoilValue()`獲得非同步的`shared state`時，可以搭配 React 的`<Suspense>`使用來製作載入效果。如果不想，Recoil也有另外提供 API : 
- `useRecoilValueLoadable()`
- `useRecoilStateLoadable()`

不同於`useRecoilValue()`回傳`ERROR`或是`PROMISE`，這兩個hooks在遇到非同步串接時會回傳含有`state`、`content`的物件：

| key/回傳內容 | 成功 | 失敗 | loading |
| --- | :---: | :---: | :---: |
| `state`     |`'hasValue'` | `'hasError'` | `'loading'` 
| `content` | 回傳的資料 | `error object` | `Promise` |


## family系列 ← 覺得很酷
可以帶入外部引數至`atom`、`selector`中的酷東東。

### atomFamily()
有時需要動態產生`atom`時，就可以使用`atomfamily()`。官方舉了一個例子來講解使用時機：
>假設你做的是一個UI編輯頁面，使用者可以動態新增元件、而且每個元件都有自己的大小、顏色、位置⋯⋯等狀態。

這時不能確定需要幾個`atom`，又不想把個別元件的樣式全部塞進一個`atom`時，應該怎麼辦～？
在[官方推薦影片（9:55-12:00）](https://www.youtube.com/watch?v=_ISAA_Jt9kI#t=9m55s)推出時，可能還沒有`atomfamily()`這個語法，為了解決問題，大叔透過帶入不重複的引數動態新增`atom`：
```jsx=
const itemWithId = memorize(id => atom({
    key: `item${id}`,
    default: { color: pink, position: {0,0} }
}))

const CanvasElement = ({elementID}) => {
  const [itemData, setItemData] = useRecoilValue(itemWithId(elementID));
  return (...);
}
```
現在Recoil提供了更方便的`atomFamily()`：
```jsx=
const itemFamily = atomFamily({
  key: 'item',
  default: {...},
});

const CanvasElement = ({elementID}) => {
  const [itemData, setItemData] = useRecoilValue(itemFamily(elementID));
  return (...);
}
```
**預設值** 除了`atom`能放的之外，還能夠放帶有參數且有回傳值的`function`：
```jsx=
const itemFamily = atomFamily({
  key: 'item',
  default: param => setColor(param),
});

```

### selectorFamily(options)
類似`selector()`，但更強大的是能帶參數到`get`、`set`中。
```jsx=
const selectColor = selectorFamily({
  key: 'selectColor',
  
  get: param => ({get}) => {
    return ... ;
  },

  set: param => ({set,get}, newValue) => {
      return ... ;
  },
});
```

`atomFamily()`的預設值可以放`function`，也可以使用`selectFamily()`：
```jsx=
const item = atomFamily({
  key: ‘item’,
  default: selectorFamily({
    key: 'colorSelect',
    get: param => ({get}) => selectColor(param);
  }),
});
```
同一個`selectFamily()`接收到相同的引數時，會回傳相同的`selector`實體。

## 最後一個
別忘了`<RecoilRoot>`！可以有複數個`<RecoilRoot>`共存，不會互相影響。但若有**巢狀的**`<RecoilRoot>`，**最內層會覆蓋外層**。

```jsx=
import { RecoilRoot } from 'recoil';

function App() {
  return (
    <RecoilRoot>
      <ComponentThatUsesRecoil />
    </RecoilRoot>
  );
}
```

## 參考資料
- [Recoil Doc](https://recoiljs.org/)
- [Recoil GitHub](https://github.com/facebookexperimental/Recoil)
- [官方推薦的影片](https://www.youtube.com/watch?v=_ISAA_Jt9kI) 
- [Redux vs Recoil](https://www.emgoto.com/redux-vs-recoil/) 
- [Recoil - a New State Management Library for React](https://www.infoq.com/news/2020/05/recoil-react-state-management/)
- [CodeSandbox Sample](https://codesandbox.io/s/recoil-selectors-set-demo-082nu?from-embed=&file=/src/App.js)

# 什麼捕獲冒泡，你是魚嗎？ 聊聊瀏覽器 DOM 的事件傳遞

身為一個前端工程師，從以前到現在的工作經驗中，凡是需要與使用者互動的，或是需要由使用者觸發的功能，總是離不開畫面上的事件處理。
這篇就要來聊聊瀏覽器中 DOM 的事件傳遞機制。

## DOM 事件

在瀏覽器的 Javascript 引擎解讀 HTML、SVG 時，會將內容分析成一個個的 [DOM (Document Object Model)](https://developer.mozilla.org/zh-TW/docs/Web/API/Document_Object_Model)，其之間的互動，是透過每一個 DOM 上面註冊的事件監聽器，去處發個別要處理的事情。

例如常見的 `onClick`、`onTouchStart`，輸入欄位的 `onInput`、`onChange`、`onBlur` 等，都是常用到的事件類型，因為太常用~~加上本人也懶~~，這邊就不贅述了。

### 事件監聽

例如大家都曾深愛的 jQuery，我們會透過這樣的方式去註冊事件監聽：

```javascript
$('#id').on('click', function(){ ... })
```

但 jQuery 已成時代的眼淚；在現代框架中，Vue 對註冊監聽器做了一些語法糖，讓你寫起來很開心：

```html
<button @click="clickHandler">click me!</button>
```

React 除了語法糖外，底層還將 DOM 事件再包過一層，並幫你全部 **代理** 到 `document` 上，效能好棒棒：

```html
<button onClick={clickHandler}>click me!</button>
```

當然，不管是什麼框架，底層都仍要透過 Javascript 進行操作：

```javascript
document.querySelector('#id').addEventListener('click', clickHandler)
```

### 事件代理

前面說到 React 會幫你把事件代理到 document 上，是什麼意思呢？

參考這邊的 [簡單小範例](https://codepen.io/GaryChu/pen/XQGxLr?editors=1010)，點擊按鈕新增 `li` 時，會一併註冊事件監聽：

```javascript
function pushHandler() {
  list.appendChild(getNewElem(list.childNodes.length))
}

function getNewElem(text) {
  const elem = document.createElement('li')
  elem.innerText = text
  elem.addEventListener('click', () => alert(text))
  return elem
}
```

這樣子很直觀，但缺點也很明顯；每新增一個元素，都會建立一個事件監聽，當數量一大，造成的記憶體消耗也將十分可觀。

再參考這個 [有事件代理的範例](https://codepen.io/GaryChu/pen/eoXQEN?editors=1010)

事件監聽註冊在外層的 `ul`，並在點擊事件觸發時判斷點到的是誰：

```javascript
function listClickHandler(e){
  if (e.target.tagName === 'LI') alert(e.target.innerText)
}
```

透過事件代理，無論內容有多少，事件監聽都只會有一組，效能好棒棒。

### 移除事件監聽

註冊事件監聽器很方便，但在確定不會再使用監聽器時，記得要透過 `removeEventListener` 將事件監聽移除啊！如果留下了無用的事件監聽器，將會造成記憶體的浪費，對效能非常的傷。

眼尖的讀者們應該注意到了，剛剛的 [簡易小範例](https://codepen.io/GaryChu/pen/XQGxLr?editors=1010) 中並沒有移除事件監聽，而且每建立一個新的子元素，都會同時建立新的函式：

```javascript
function getNewElem(text) {
  const elem = document.createElement('li')
  elem.innerText = text
  elem.addEventListener('click', () => alert(text)) // 這裡建立了新的匿名函式！
  return elem
}
```

比較好的寫法，應該要將匿名函式抽出來，並在移除子元素時一併移除事件監聽器：

```javascript
function popHandler() {
  const elem = document.querySelectorAll('#list>li')[list.childNodes.length - 1]
  elem.removeEventListener('click', alertText) // 移除事件監聽
  elem.remove()
}

function getNewElem(text) {
  const elem = document.createElement('li')
  elem.innerText = text
  elem.addEventListener('click', eventHandler)
  return elem
}

function eventHandler(e) {
    alert(e.target.innerText)
  }
```

在 Vue & React 等框架中，只要是使用內建的語法註冊事件監聽，它們都會自動在無用的時候移除，使用時可以放心；不過如果是自己撰寫事件監聽，務必要記得移除喔。

## 捕獲 & 冒泡

好啦跑題太久了，所以到底什麼是捕獲 & 冒泡？

根據 [W3C 所定義的 Event Flow](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)：
 [![DOM Event Architecture](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

瀏覽器中的事件傳遞過程分成三個階段：

- 捕獲階段：DOM 樹的最外層依序向內，過程觸發個別元素的捕獲階段事件監聽
- 目標階段：到達事件目標，[依照註冊順序觸發事件監聽](https://developer.mozilla.org/zh-TW/docs/Web/API/EventTarget/addEventListener)
- 冒泡階段：由事件目標依序向外，過程觸發個別元素的冒泡階段事件監聽

這也就是剛剛提到的 **事件代理** 所利用的機制了；在事件傳遞過程中，捕獲 & 冒泡階段必然會經過外層元素，因此可以將事件監聽註冊到外層元素上。

另外，當我們在使用 `addEventListener` 註冊事件監聽器時，可以傳遞第三個參數，指定這個事件要在什麼階段觸發：

```javascript
elem.addEventListener('click', eventHandler) // 未指定，預設為冒泡
elem.addEventListener('click', eventHandler, false) // 冒泡
elem.addEventListener('click', eventHandler, true) // 捕獲
elem.addEventListener('click', eventHandler, {
  capture: true // 是否為捕獲。IE、Edge 不支援。 其他物件屬性請參考 MDN
})
```

如上圖所示，一個 DOM 事件發生時，會依序由最外層的 `window` 開始依序向內傳遞事件，一直傳到我們的事件目標，觸發完目標上註冊的事件監聽，再進入冒泡階段反向傳遞；藉由指定觸發的階段，就能確定執行的順序...了吧？

## Bug

前陣子在工作上遇到一個奇妙的 Bug，原本以為是 Vue 的 Directive 造成的 Bug，還跑去開 issue，結果發現是跟 DOM 的事件傳遞機制有關，也就因此有了這篇。

可以參考：

- [1. 用 Vue 模擬狀況](https://codepen.io/GaryChu/pen/EJYRKz?editors=1111)
- [2. vue issue](https://github.com/vuejs/vue/issues/9794)
- [3. 用 Javascript 模擬狀況](https://codepen.io/GaryChu/pen/XQGOqQ?editors=1011)

在 3 的狀況模擬內，註冊了三個監聽器：

```javascript
const content = document.getElementById('content')
const box = document.getElementById('box')

content.addEventListener('click', () => {
  content.removeEventListener('click', bubblingHandler)
  content.removeEventListener('click', capturingHandler)
  box.parentNode.removeChild(box)
}, false)

content.addEventListener('click', bubblingHandler, false)
content.addEventListener('click', capturingHandler, true)

// ">>> capturing"
```

如果目標階段是 **完全依照註冊順序執行**，console 應該不會印出東西吧？
但如果調換註冊的順序如下：

```javascript
content.addEventListener('click', bubblingHandler, false)
content.addEventListener('click', capturingHandler, true)

content.addEventListener('click', () => {
  content.removeEventListener('click', bubblingHandler)
  content.removeEventListener('click', capturingHandler)
  box.parentNode.removeChild(box)
}, false)

// ">>> bubbling"
// ">>> capturing"
```

則可以看到先後分別印出了 **冒泡階段** 及 **捕獲階段** 的文字；這樣又證明確實是依照註冊順序執行沒錯。

![黑人問號](https://i.imgur.com/kTIcVI4.jpg)

好吧我真的搞不懂...

## 總結

以上就是這次關於 DOM 事件傳遞的分享。如果對於內文有任何問題，或是有錯誤的地方，都歡迎您的回應。也非常歡迎您對這個讓我困惑許久的 Bug 提出您的觀點～

## 參考資料

- [MDN: DOM (Document Object Model)](https://developer.mozilla.org/zh-TW/docs/Web/API/Document_Object_Model)
- [MDN: Event​Target​.add​Event​Listener()](https://developer.mozilla.org/zh-TW/docs/Web/API/EventTarget/addEventListener)
- [DOM 的事件傳遞機制：捕獲與冒泡](http://huli.logdown.com/posts/2223612-dom-event-capture-and-propagation)
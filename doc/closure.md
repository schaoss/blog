# 你一直在用，但從沒搞懂的閉包

![Closure](https://i.imgur.com/KBG48Hw.png)

Javascript 是個有趣的語言，擁有幾個獨特的特性；這篇我們就來聊聊實務中非常常見，但又總是讓人搞不太懂的 **閉包（Closure）**

## 範例

直接程式碼出發吧，首先考慮下面這段簡單的程式碼：

```javascript
function add(num) {
  function func(x = 0) {
    return num + x
  }
  return func
}

let addFive = add(5)
console.log(addFive(8)) // 13
```

上例中，如果只看 `add` 回傳的 `func` 函式，會發現其自身並不存在 `num` 這個變數，而是找到了外層 `add` 函式中的傳入的參數 `num`；雖然 `add` 函式已經執行完成，但內部的 `num` 變數卻因為還有被 `func` 呼叫，沒有跟著函式使用完畢而消失。

再看一個實際應用的例子：

```javascript
loadPicture() {
  let count = this.pageContent.length
  const load = (target, resolve) => {
    const counter = () => (--count ? count : resolve())
    let img = new Image()
    img.onload = counter
    img.onerror = counter
    img.src = target.url
  }
  return new Promise(resolve => {
    if (!count) return resolve()
    this.pageContent.forEach(target => load(target, resolve))
  })
}
```

這段程式碼是節錄自目前工作中處理預載圖片部分的程式碼，也用到了閉包：

- 一開始建立的 `count` 變數，在 `new Promise()` 中的匿名函式，及 `counter` 函式中被直接呼叫。
- `load` 函式接收了最後回傳的 `new Promise()` 中匿名函式的 `resolve`，並在 `counter` 中直接呼叫，在圖片的 `onload` 或 `onerror` 時觸發。

即使函式已經執行結束，其內部的變數卻並未跟著消除，還能繼續被呼叫；這種能將外層變數「包」在內層暫存、使用的方式，就是所謂的「閉包」。

## 說明

那麼，閉包是怎麼發生的呢？那就要聊到所謂的 **執行環境** 了。

### 執行環境

**執行環境（Execution Context，EC）** 指的是 Javascript 底層在程式執行時，所建立的一個物件，主要是儲存了：

- 內部的變數（Variable Object，VO；這也就是 **Hoisting** 發生的原因！）
- 外部環境（也就是 **作用域鏈**）
- 這個環境的 `this` 值。

![Execution Context](https://i.imgur.com/Z7ts45M.png)

例如下面的例子：

```javascript
var a = 0

function b() {
  var a = 10
  function c() {
    console.log(a)
  }
  c()
}

b() // 10
```

當在執行的時候，可以想像 EC 會形成類似下圖的狀態：

![EC 1](https://i.imgur.com/tJU7Jmd.png)

依序會進行以下步驟：

- 準備階段
  - 建立 `Global EC` ，預留了變數 `a` 及函式 `b` 的空間
  - 準備函式 `b`，建立 `function b() EC`，預留了區域變數 `a` 及函式 `c` 的空間
  - 準備函式 `c`，建立 `function c() EC`，並依照 **Scope Chain** 找到 `function b() EC` 中的變數 `a`

- 執行階段
  - 變數 `a` 賦值 `a = 0`，接著叫函式 `b()`
  - 執行 `b()`，區域變數 `a` 賦值 `a = 10`，接著叫函式 `c()`
  - 執行 `c()`，呼叫 `console.log(a)`；印出 `10`
  - `c()` 執行結束，消除 `function c() EC`
  - `b()` 執行結束，消除 `function b() EC`
  - 程式執行結束

依照呼叫的順序，依序建立 EC，並在執行完成時將 EC 消除，釋放記憶體空間。

那麼 **閉包** 發生時，EC 會有什麼改變呢？

稍微修改一下剛剛的例子，讓函式 `b` 改成回傳函式 `c`：

```javascript
var a = 0

function b() {
  var a = 10
  function c() {
    console.log(a)
  }
  return c
}

var func = b()
console.log(a) // 0
func() // 10
```

![EC 2](https://i.imgur.com/p9SDQH4.png)

- 準備階段
  - 建立 `Global EC` ，預留了變數 `a`、`func` 及函式 `b` 的空間
  - 準備函式 `b`，建立 `function b() EC`，預留了區域變數 `a` 及函式 `c` 的空間
  - 準備函式 `c`，建立 `function c() EC`，並依照 **Scope Chain** 找到 `function b() EC` 中的變數 `a`

- 執行階段
  - 變數 `a` 賦值 `a = 0`，變數 `func` 賦值成函式 `b()`
  - 執行 `b()`，區域變數 `a` 賦值 `a = 10`，回傳函式 `c` 給 `func`
  - `b` 執行結束，消除 `function b() EC`；
  > 執行到這邊時，`function b() EC` 內的 `a` 被回傳的函數 c 閉包了！
  - 執行 `console.log(a)`，印出 `Global EC` 中的 `a`：`0`
  - 執行 `func()`，呼叫 `console.log(a)`；印出 `10`
  - `func()` 執行結束，消除 `function c() EC`
  - 程式執行結束

由於閉包，在 `b()` 執行結束時，其中的變數 `a` 並未跟著 EC 一起消失，而是留給了 `function c() EC` 的參照使用，直到參照消失，其佔用的記憶體才會跟著一起釋放。因此在使用閉包時需要注意，冗餘的閉包只會造成記憶體的負擔！

### 透過 let, function, catch 建立新的 EC

最後來看一個經典的面試題：

> 間隔一秒，依序印出數字 1~5

```javascript
for(var i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log(i)
  }, 1000 * i)
}
// 間隔一秒印出五次 6
```

程式並未如預期的依序印出 `1 ~ 5`，而是五個 `6`，為什麼會這樣呢？

由於透過 `var` 宣告的變數 `i`，不會產生新的執行環境，`i` 就被閉包到 `setTimeout` 中的匿名函式，並停在迴圈結束的狀態 `i = 6`，所以印了五次 `6`。

修改方式也很簡單，只要建立新的 `function`將 i 當下的值留下來就可以囉：

```javascript
for(var i = 1; i <= 5; i++) {
  (function(i) {
    setTimeout(function() {
    console.log(i)
  }, 1000 * i)
  })(i)
}
// 間隔一秒依序印出 1, 2, 3, 4, 5
```

ES6 後更是只要將 `var` 換成 `let` 就輕鬆解決啦：

```javascript
for(let i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log(i)
  }, 1000 * i)
}
// 間隔一秒依序印出 1, 2, 3, 4, 5
```

還有一種比較玄妙的用法，透過`catch`建立新的 EC：

```javascript
for(var i = 1; i <= 5; i++) {
  try {
    throw i
  } catch (i) {
    setTimeout(function() {
      console.log(i)
    }, 1000 * i)
  }
}
// 間隔一秒依序印出 1, 2, 3, 4, 5
```

聽說在 ES6 之前，有些套件是透過這樣建立 EC 的，但現在可千萬別這樣用啊 XD

## 總結

閉包是 Javascript 最有趣的特性之一，直接牽扯到了底層 EC 的建立與釋放；我們從閉包出發，透過幾個範例稍稍了解了 EC 的概念，未來如果遇到像是 Hoisting、`this` 的指向...等等，令人同樣稍感困惑的 Javascript 特性，也會變得沒那麼難懂喔！

以上就是這次的 Closure 介紹，如果看完有任何感想，歡迎你在下面留言回應；如果文中有不清楚、不懂、錯誤的地方，也都歡迎你給予回應指教，教學相長。
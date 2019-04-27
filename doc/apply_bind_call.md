# 一次搞懂前端面試最愛問的 Javascript apply、bind、call

數年前在四處求職面試時，時常遇到面試官詢問 Javascript 中 Function 的 `apply`、`bind`、`call` 差異是什麼，但當時的我僅一知半解，回答得支支嗚嗚；前陣子開始練習刷 CodeWars 的題目，便時常遇到這些函式的應用，也有了機會重新深入理解這三者的差異。

## `apply`、`bind`、`call` 是什麼？

Javascript 中萬物皆物件，除了數種基本型態之外，其他常用的 `Function`、`Array`、`Date` 等等，其實都是 Runtime 中預設的 `Object`，Runtime 會建立好個別預設的 `prototype`，並在物件實例產生時，把 `__proto__` 指到個別的 `prototype`，使物件實例可以使用 `prototype` 中定義好的屬性、函式；這也就是常聽到的 **原型鏈**。

例如：
```javascript
let arr = new Array()
arr.__proto__ === Array.prototype // true
```

而這篇文章要討論的 `apply`、`bind`、`call`，則是 `Function.prototype` 中的三個函式，因為他們之間有些相似，因此時常被拿來一起討論、比較。

## bind

首先來看看可能比較多人使用過的 `bind`。

大家可能對綁 `this` 比較熟悉，例如：

```javascript
var name = 'foo'
function logName(){
  console.log(this.name)
}
var tmp = {
  name: 'bar'
}
var newFunction = logName.bind(tmp)

logName() // foo
newFunction() // bar
```

透過 `bind()`，借用已建立的函式來創造新的函式，但將 `this` 綁到指定的物件上。

比較少使用到的是，`bind` 還可以綁定傳入函式的參數。例如：

```javascript
function sum(a,b) {
  return a + b
}
let plusTwo = sum.bind(this, 2)
let plusFive = sum.bind(this, 5)

sum(2, 3) // 5
plusTwo(7) // 9
plusFive(1) // 6
```

藉由 `bind` 固定傳入函式的參數，增加了函式複用的彈性，熟悉使用的話真的很方便！

總結一下，`bind()` 接受的第一位參數為指定的 `this`，其餘參數則依序傳給被綁定的函式，作為固定的參數，最後會回傳一個新的函式。

## apply & call

接著來看 `apply` 跟 `call`，因為兩者的行為幾乎一模一樣，僅接受的參數類型不同，這邊就一起說明。
不囉嗦，直接看個例子：

```javascript
let str = '12345'
Array.prototype.map.apply(str, [c => c ** 2])
// [1, 4, 9, 16, 25 ]
Array.prototype.map.call(str, c => c ** 3)
// [1, 8, 27, 64, 125 ]
str.map(c => c ** 2)
// TypeError: str.map is not a function
```

`apply` 跟 `call`，可以想像成跟別的物件借用函式。第一參數同樣為 `this`，其餘參數則為傳入函式中的參數。例如前例中跟 `Array.prototype` 借用 `str` 自己無法呼叫的 `map` 函式。

而`apply` 與 `call` 唯一的差別，就是 `apply` 接受的要傳入函式參數為陣列，`call` 則為逐項傳入。

再看一個簡單的例子：

```javascript
function sum (...argu) {
  return argu.reduce((acc,c) => +c == c ? acc + c : c)
}
let params = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum.apply(null, params)   // 55
sum.call(null, ...params) // 55
```

ES6 開始有了展開運算子，`call` 跟 `apply` 在使用時差異幾乎可以忽略不計了。不過網路上有些討論說 `call` 的效能略優於 `apply`，筆者沒有實際測試，大家可以參考看看。
> a for Array, c for Comma

## 總結

以上就是這次關於 apply、bind、call 的心得分享，熟悉之後活用在專案中吧！如果文中有任何不清楚或錯誤的地方，也都歡迎你不吝告知！

## 參考資料

 - [CodeWars: 5 kyu - Javascript Magic Function](https://www.codewars.com/kata/55caf9ff29c318407c00001f)
 - [MDN: Function​.prototype​.apply()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
 - [MDN: Function​.prototype​.bind()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
 - [MDN: Function​.prototype​.call()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
 - [apply 与 call 性能分析](https://www.jianshu.com/p/f90a5965916b)
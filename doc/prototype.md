# Javascript 是個真正的「物件」導向程式語言

前陣子看了部門面試前端新人的題目，其中一題在考面試者對 Javascript Prototype 的認識；這篇就讓我們來好好認識一下 Javascript 的型別與 Prototype。

## 基本型別

Javascript 是 [弱型別](https://zh.wikipedia.org/wiki/%E5%BC%B7%E5%BC%B1%E5%9E%8B%E5%88%A5) 語言，所有的資料都會在這些基本型別中轉換；在 Javascript 中一共有六種基本型別，其中最常見、最好懂的應該是這三者：

- `boolean`：就是真假值、`true` 或 `false` 啦。
- `number`：數字，有關數字的都找他。
- `string`：字串，就是一串文字。

接下來的這兩位對剛學 Javascript 的新手來說，應該是最常搞不清楚的吧：

- `undefined`：未定義，想像成是變數的 placeholder。
- `null`：空，其實是一個內建的特製 **物件**。

```javascript
let a
console.log(a) // undefined
a = null
console.log(a) // null
console.log(typeof a) // object
```

`undefined` 跟 `null` 設計上的用法不同，也隱含著不同的語意，使用時應該要把明確指定是空值的東西賦值成 `null`，並且極力避免把任何東西賦值成 `undefined`；保留好未定義的語意，幫自己跟同事節省除錯的時間。

另外一種基本型別，是 ES6 新增的 `Symbol`，用來建立一個匿名且獨一無二的值，有興趣深入理解的讀者可以參考 [MDN 的介紹](https://developer.mozilla.org/en-US/docs/Glossary/Symbol)，這邊就不贅述了。

最後一種基本型別就是 `object` 了。

## 你才 Object，你全家都 Object

對，Javascript 是真的全家都 Object。

~~我就突然很想要寫這句，我很抱歉 XDDD~~

例如常用的 `Math`，是一個包裹著數學相關功能的內置物件、`console` 則是含有許多工具函式，供工程師開發使用，例如：`console.log('print something')`。

另外有一群物件，如同前面提到的 `null` 一樣，是在底層被特別處理過的物件，如 `Array`、`RegExp`、`Function` 等，除了擁有眾多功能函式外，還從底層就支援了 **物件實體語法（object literal）**：

```javascript
let arr1 = new Array(0, 1, 2, 3, 4)
let arr2 = [0, 1, 2, 3, 4]

let reg1 = new RegExp('bar')
let reg2 = /foo/

let func1 = new Function('num', 'return num * 2')
let func2 = function(num) { return num * 2 }

console.log(typeof arr2) // object
console.log(typeof reg2) // object
console.log(typeof func2) // **function**
```

### Function

在前述的眾多預設物件中，最特別、最核心，也最需要理解的，莫過於 `Function` 了。

`Function` 在 Javascript 中是很特別的存在，是個有預設屬性、可呼叫、還會建立 [執行環境](https://stackoverflow.com/questions/9384758/what-is-the-execution-context-in-javascript-exactly) 的物件

> 在 [之前關於閉包的文章中](./closure.md) 也有提到 Function 會建立執行環境的特性，還沒看過的讀者可以點過去看看，讓你更了解閉包及執行環境喔！

例如前面舉例的 `func1`，他擁有 `name: "func1", length: 1` 等預設屬性、可以被執行，也會建立自己的執行環境；但要注意的是，透過 `new Function()` 建立的函式，EC 會建立在 Global EC 上：

```javascript
let x = 'foo'
let a = function() {
  let x = 'bar'
  let func1 = function() {
    console.log(x)
  }
  let func2 = new Function('console.log(x)')

  func1() // bar
  func2() // foo
}
```

另外，也因為建立 `Function` 牽扯到底層的 EC 建立，透過 `new Function()` 或 `eval()` 動態建立的函式，在效能表現上勢必會比一般的函式差，還有安全性上的潛在風險，一般的情況中應該要盡量避免這樣動態建立函式喔！

## prototype

說明完基本型別，該來說明這個~~文章農場~~標題了。

Javascript 是個 [**基於原型（Prototype）的物件導向程式語言**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Details_of_the_Object_Model)。不同於如 Java、C# 等 **基於類別（Class）的物件導向程式語言**，透過定義 Class、指定繼承來傳遞屬性及函式；在 Javascript 中會預先建立好 **物件原型**，並在新物件建立時，指定他的 **原型** 要參照到哪個物件：

```javascript
function Person(name) {
  this.name = name
  this.hello = function() {
    console.log(`Hi, I'm ${name}`)
  }
}
Person.prototype = new Person

let john = new Person('John', 20)

console.log(john)
// age: 20
// hello: ƒ ()
// name: "John"
// __proto__: Person

console.log(john.__proto__)
// age: undefined
// hello: ƒ ()
// name: undefined
// __proto__: Object
```

上例中，我們建立了新的原型 `Person`，並將原型指定為自己建立的新物件；接著透過 `new` 運算子，建構一個原型參照 `Person` 的新物件 `john`。

可以從 `console.log` 觀察到，`john` 中包含了建構式中的指定的所有屬性及函式，並擁有一個新的屬性 `__proto__`，參照到了建構他的物件 - `Person`，再看看 `john.__proto__` 中的 `__proto__` 屬性，參照到了 `Object`；這種一層一層往上參照的物件模型，也就是俗稱的 **原型鏈**，是 Javascript 這類基於 Prototype 的物件導向程式語言中，至關重要的機制。

如果再繼續沿著原型鏈往上找，你會找到 `Object` 的物件原型，最後找到一切的原型 -  `null`：

```javascript
console.log(john.__proto__.__proto__)
// constructor: ƒ Person(name, age)
// __proto__: Object

console.log(john.__proto__.__proto__.__proto__)
// constructor: ƒ Object()
// hasOwnProperty: ƒ hasOwnProperty()
// isPrototypeOf: ƒ isPrototypeOf()
// propertyIsEnumerable: ƒ propertyIsEnumerable()
// toLocaleString: ƒ toLocaleString()
// toString: ƒ toString()
// valueOf: ƒ valueOf()
// __defineGetter__: ƒ __defineGetter__()
// __defineSetter__: ƒ __defineSetter__()
// __lookupGetter__: ƒ __lookupGetter__()
// __lookupSetter__: ƒ __lookupSetter__()
// get __proto__: ƒ __proto__()
// set __proto__: ƒ __proto__()

console.log(john.__proto__.__proto__.__proto__.__proto__)
// null
```

另外，如果我們在 `Person` 原型中加入新的屬性，會發生什麼事呢？

```javascript
console.log(john.newParam) // undefined
Person.prototype.newParam = 'new!'

console.log(john.newParam) // new!

let bob = new Person('Bob', 46)
console.log(bob.newParam) // new!
```

可以看到，如果更動了物件原型，無論是已經存在的，或是原型變動後才新建的，整條原型鏈的物件都會因此跟著變動；在實務上應該要盡量避免處理原生物件的物件原型，以避免影響其他模組套件；如果真的不得已非棟不可，也請務必要小心。

### Class

最後提一下 ES6 新增的 `Class`，有寫過 React 的讀者應該對類似 Class 的語法不陌生：

```javascript
class Welcome extends React.Component {
  constructor(props) {
    super(props)
    this.state = { counter: 0 }
  }
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

雖然他看起來、寫起來，都很像是 Java、C# 等語言中的「類別」，但其實 [他只是 Prototype 的語法糖](https://medium.com/enjoy-life-enjoy-coding/23e4a4a5e8ed)，底層一樣是透過 Prototype 進行物件的定義與繼承。

## 結語

回到標題：Javascript 是個真正的「物件」導向程式語言；會這樣說，是因為 Javascript 中幾乎一切都是物件！不同於基於類的程式語言，在基於原型的語言設計中，透過「物件原型」定義了物件模板，透過「原型鏈」，一層一層的參照物件原型，實現了屬性及函式的傳遞；另外，也因為一切都是參照到物件原型，使用時要注意避免對物件原型的變動，以預防牽一髮動全身的大災難。

希望透過這篇文章，能夠讓讀者您對 Javascript 中的物件類型、Prototype 有一些基本概念。如果看完後有任何心得，都歡迎您在底下回應，一起討論；若文中有不清楚或錯誤的地方，也請您不吝告知～
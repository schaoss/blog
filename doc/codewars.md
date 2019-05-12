# 工程師們刷題刷起來 - CodeWars 刷題心得分享

這一年來一直有斷斷續續的在 [CodeWars](https://www.codewars.com/) 上練習解題，也就是俗稱的刷題，最近這幾個月玩的比較認真，終於達到了 1000 分的小目標，於是想稍微整理一下這陣子的心得和大家分享。

## CodeWars 是什麼？

CodeWars 就是一個類似 [LeetCode](https://leetcode.com/) 的程式練習網站，但結合了經驗值、等級、戰隊、排行榜等許多遊戲化的要素，讓你刷題不無聊。

### 與 LeetCode 的比較

刷題時最常被同事問到的就是這個。

同樣是刷題網站的兩者相比較起來，LeetCode 的題目來源主要是蒐集是各大科技公司的 **面試題目**；且 LeetCode 會在題目完成後，告知你提交的程式效能如何、使用的記憶體、運行時間贏過全部挑戰者中的多少百分比，是比較重視「演算法」的練習平台。

而 CodeWars 則開放讓所有使用者自創題目，再由其他玩家協同開發各語言版本、提供 Test case、回報 issue 等等；並在完成挑戰之後可以看到別人送出的程式，並給予評論，也能藉由理解別人的想法來再次看到自己的盲點。

對我來說，LeetCode 比較嚴肅，如果有要去大公司面試，好像真的要把上面的經典題目刷過一輪比較保險；CodeWars 相對 LeetCode 就多元有趣了不少，挑戰不同領域會遇到的實際題目，下班、周末沒事來刷個一兩題當遊戲也是很棒的練習，於是就在這邊一直玩下去了 XD

## 印象深刻的題目

好啦我知道前面字太多了，該來點程式了；我大部分的解題過程都丟在 [這裡](https://github.com/schaoss/HappyCodeWars)，想參考我玩過的題目可以在那逛逛。

> Talk is cheap. Show me the code. - Linus Torvalds

### Connect Four

[原始題目連結](https://www.codewars.com/kata/56882731514ec3ec3d000009)

相信大家多少都玩過 [四連環](https://en.wikipedia.org/wiki/Connect_Four) 這款經典的益智對戰遊戲；這題要實作出四連環的獲勝者判斷機制，可以說是非常輕鬆且貼近生活的題目。

![Connect Four](https://upload.wikimedia.org/wikipedia/commons/a/ad/Connect_Four.gif)

題目會給予一段資料陣列，內容就依序是參加者各別的下子位置，要求的輸出則是遊戲贏家；我的想法是要先設計遊戲機制的邏輯，也就是落子到棋盤的這部分，接著在每次落子時加上勝負判斷的機制。

```javascript
// < 4 kyu > - Connect Four
// https://www.codewars.com/kata/connect-four-1

function whoIsWinner(input) {
  // 棋盤
  const grid = new Array(7).fill(0).map(() => [])
  let winner = 'Draw'

  // 依序落子
  for (let i = 0; i < input.length; i++) {
    let [char, color] = input[i].split('_')
    let row = char.charCodeAt() - 65

    // dropItem
    grid[row].push(color)
    // check winner
    if (i > 6 && check(row, color)) {
      winner = color
      break
    }
  }
  return winner

  // 判斷勝負
  function check(i) {
    let j = grid[i].length - 1
    if (checkV() || checkH() || checkS() || checkRS()) {
      return true
    }
    return false

    // 垂直
    function checkV() {
      if (j > 2) {
        return grid[i][j] === grid[i][j - 1] && grid[i][j - 1] === grid[i][j - 2] && grid[i][j - 2] === grid[i][j - 3]
      }
      return false
    }

    // 水平
    function checkH() {
      for (let k = 0; k < 4; k++) {
        if (
          grid[k][j] &&
          grid[k + 1][j] &&
          grid[k + 2][j] &&
          grid[k + 3][j] &&
          grid[k][j] === grid[k + 1][j] &&
          grid[k + 1][j] === grid[k + 2][j] &&
          grid[k + 2][j] === grid[k + 3][j]
        ) {
          return true
        }
      }
      return false
    }

    // 斜向 ↗
    function checkS() {
      for (let x = i - Math.min(i, j), y = j - Math.min(i, j); x < 4 && y < 3; x++, y++) {
        if (
          grid[x][y] &&
          grid[x + 1][y + 1] &&
          grid[x + 2][y + 2] &&
          grid[x + 3][y + 3] &&
          grid[x][y] === grid[x + 1][y + 1] &&
          grid[x + 1][y + 1] === grid[x + 2][y + 2] &&
          grid[x + 2][y + 2] === grid[x + 3][y + 3]
        ) {
          return true
        }
      }
      return false
    }

    // 反斜 ↖
    function checkRS() {
      for (let x = i + Math.min(6 - i, j), y = j - Math.min(6 - i, j); x > 2 && y < 3; x--, y++) {
        if (
          grid[x][y] &&
          grid[x - 1][y + 1] &&
          grid[x - 2][y + 2] &&
          grid[x - 3][y + 3] &&
          grid[x][y] === grid[x - 1][y + 1] &&
          grid[x - 1][y + 1] === grid[x - 2][y + 2] &&
          grid[x - 2][y + 2] === grid[x - 3][y + 3]
        ) {
          return true
        }
      }
      return false
    }
  }
}

```

主要的難點大概是勝負判斷的部分，要考慮各個方向判斷邏輯微妙的不同。最後我將各個方向的判斷各自拆出分別實作，降低一點點複雜度。

### Javascript Magic Function

[原始題目連結](https://www.codewars.com/kata/55caf9ff29c318407c00001f)

這題是個活用 Javascript 特性的巧妙題目。要求寫出一個神奇的 Function，可以接受任意數量的參數、被連續呼叫任意次、且會回傳所有參數數字的總和。

由於可以被呼叫任意次，因此一定要回傳 Function；但當在進行比較時，卻要能回傳出正確的數字。乍看之下好像不太可能，但如果理解 Javascript 中「一般相等」的運作機制，好像就能找到突破口了。

```javascript
function MagicFunction(...argu) {
  // 參數加總
  let sum = argu
    .filter(a => !isNaN(a))
    .map(Number)
    .reduce((acc, c) => acc + c, 0)
  
  // 透過 bind，將加總帶到新函式中
  let func = MagicFunction.bind(this, sum)
  // 改寫函式的 valueOf -> 在一般相等比較時回傳參數的加總
  func.valueOf = () => sum
  // 回傳函式 -> 可以連續呼叫
  return func
}
```

## 目前成果

稍微比較了我自己以前跟現在的工作習慣與題目解法，整理了自己有感的幾點成長：

### 更熟悉語言原生特性

在與陣列有關的地方，我發現自己變得非常喜歡用原本不太熟悉的 `array.reduce` 來解決事情，能夠繞完一圈就把資料全部整理乾淨，真的很方便！另外也對於 `Function.prototype`、函式中的 `arguments` 等特性有了更多認識，並且趁機搞懂了 [前端面試最愛問的 Javascript apply、bind、call](./apply_bind_call.md)。

還有一般寫程式時會被要求盡量避免甚至禁止的 `eval`，在特定的情況也會有奇效；能夠把字串處理後的結果轉回程式再執行，又是一種完全不同的思考方向。還有像前面提到的一般相等、絕對相等之間的差異，自動轉型的運作機制，也都是在解題過程中逐步摸索出來的。

藉由刷題，也能稍微熟悉了這些在目前的工作中比較少用到的語言特性，一點一點的增加自己基礎知識的儲備量。

### 練習 Functional Programming

因應一部分題目的要求，也開始對 Functional Programming 的核心概念有了理解。例如純函數、 Higher order Function、[Curry 化](https://zh.wikipedia.org/zh-tw/%E6%9F%AF%E9%87%8C%E5%8C%96) 函數、Compose 組合函數、鍊式函式串接等等。

現在遇到關於字串處理的問題，可能就會實作成陣列 & 字串的函式串接。
像是 [這題](https://www.codewars.com/kata/5411c4205f3a7fd3f90009ea) 我的解法：

```javascript
const stringParse = s => typeof s !== 'string' ? 'Please enter a valid string' 
  : s.length < 3 ? s 
  : s.split('')
     .reduce((acc, c, i, arr) => (c === arr[i - 1] ? [...acc, c] : [...acc, '|', c]))
     .join('')
     .split('|')
     .map(c => (c.length > 2 ? `${c.substring(0, 2)}[${c.substring(2)}]` : c))
     .join('')
```

透過不同的思考模式，練習不同的撰寫風格也是一種很棒的成長。

### 各種玄妙技巧

由於 CodeWars 提交答案後看得到別人的寫法；即使是在相對簡單的題目中，也可以見識到各路大神們腦洞大開的解法，並從中學到不少奇妙的小技巧。

例如用 `~~` 取代 `parseInt()`，更短的寫法、更好的效能，但更 **低** 的可讀性：

```javascript
parseInt('456.123')
// 456
~~'456.123'
// 456
```

也可以透過 **逗號** 串接多個命令句並回傳最後一個的結果：

```javascript
function awesome(arr) {
  return (n = arr.length), arr[n - 1] ** 2
}
```

其他還有在連續乘 2 除 2 的地方可以活用 [左移 & 右移](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Guide/Expressions_and_Operators#%E4%BD%8D%E5%85%83%E9%81%8B%E7%AE%97%E5%AD%90)，連續布靈值的合併可以用 AND、OR、XOR 運算子等等。

雖然這些寫法不一定實用，但理解的過程也是蠻有趣的對吧？

## 總結

在練習的過程，一步一步感受到自己的成長與進步，真的是很有成就感的事情；終於達到 1000 分的當下真的是超感動的。如果你也是個想要透過持續練習來讓自己進步的工程師，又不喜歡一直練習冷冰冰的演算法，我想 CodeWars 是個開心、有趣又不失成效的練習平台。

以上就是近期的刷題心得，等我哪一天能升到 kyu 1 時再來續集吧～如果對於內文有任何問題，或是文中有錯誤、不清楚的地方，都歡迎您回應討論。
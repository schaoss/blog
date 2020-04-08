# 透過 MJML 快速完成 RWD Email 排版

去年的 [前端三十系列文](https://medium.com/schaoss-blog/36293ebcffe7) 中，我寫道 **「2019 年了，不要再用 Table 排版了」**；PO 文的幾天後，我卻在工作中被主管指派要切版新產品的訂單通知信，並實作專門產出 Email Template 給各產品使用的內部 Service，難道這就是說錯話的現世報嗎？不過，我也因此開始研究那被我棄如敝屣的 Table 排版，並踏入了 RWD Email 排版的大坑。

## Email 排版的困境

為什麼 RWD Email 排版是大坑呢？主要是 CSS 支援度的問題；就如同各家瀏覽器間微妙的差異，各 Email Client 對 CSS 屬性及 HTML 標籤的支援度更是嚴重的参玼不齊令人驚訝的低。

也如同前端工程師在使用支援度不高的由屬性會查閱 [Can I use](https://caniuse.com/) 一樣，關於 Email 排版限制的細節，可以參考 [Can I email](https://www.caniemail.com/)；其中我認為主要的痛點應該有以下幾點：

### inline style

在部分的 Email Client 中，是無法使用 `<style>` 的，也就是無法定義共用的 CSS 樣式。開發者在進行 Email 切版時，只能將自訂的樣式寫成 inline style 的形式，方能套用到指定元素上，即使 定義了產品風格的基礎樣式，仍需要不斷地複製貼上到各元素中，HTML 的部分就會變得相對複雜且難以維護。

### 技術限制

除了前述的 `<style>` 外，還許多基本的 CSS 屬性，在部分 Email Client 中也是無法正確呈現的；例如：
- `float`
- `position`
- `opacity`
- `border-radius` 
- `:hover`
- `padding` 只能在 `<td>` 上正確呈現
- `margin` 會吃到 background 的設定

另外，一些相對新穎華麗的屬性更是不用想了，幾乎全部 Email Client 都不支援：
- `display: flex`
- `display: grid`
- `box-shadow`
- `filter`
- `var()`
- `calc()`

種種的技術限制，除了工程師切版時會感到痛苦萬分之外，同時也大幅壓縮了設計的可能性，使得 Email 的版面往往都設計的相對單純，以避免實際信件中有著不預期的呈現效果。

### table 排版

由於 `float`、`position`、`display: flex`、`display: grid` 都無法使用，且 `margin` 的行為不如預期、`padding` 又只能在 `<td>` 上正確呈現，開發者如果想要確保樣式在各個 Email Client 都能正確呈現，幾乎就只能透過 `<table>` 來實作排版；各種玄妙又恐怖的巢狀 `<table>` 也就因此誕生了。

> 沒有玩過 `<table>` 排版，想觀摩的人，可以參考這篇教學：[從頭開始構建一個 HTML Email 模板](https://webdesign.tutsplus.com/zh-hant/articles/build-an-html-email-template-from-scratch--webdesign-12770)

### RWD

如同前述，大部分常見的排版屬性都無法使用，如果想要在 Email 中實作 RWD 效果，幾乎只能透過設定固定的寬度，仰賴元素自然流動來達成，元素的對齊方式也十分受限。

說了這麼多 Email 排版的痛點，有沒有合適的工具可以解決這些問題呢？

## MJML

[MJML](https://github.com/mjmlio/mjml) 是針對 RWD Email 特殊的規格而設計的標記式語言（markup language），開發者只需要跨過極低的進入門檻，稍微認識 MJML 自定義的標籤，便能快速寫出 RWD Email。

### 使用方式

MJML 提供了 [線上開發/預覽工具](https://mjml.io/try-it-live)、[桌面版應用程式](https://mjmlio.github.io/mjml-app/)、[npm 套件](https://www.npmjs.com/package/mjml) 等多種使用方式，開發者可以視自身習慣與任務需求，來決定開發方式；另外，MJML 也提供了 [各式 IDE 的 MJML Plugin](https://mjml.io/documentation/#applications-and-plugins)，讓開發者能即時得到語法提示，開發者體驗豪棒棒！

### 特色

MJML 可以有效的協助開發者降低 Email 切版的痛苦，並加速開發；除了 Email Client 的語法限制無法處理，前面提到 Email 排版的其他幾項痛點，MJML 都自動幫開發者處理好了。

例如 Email 必須撰寫 inline style 的問題，MJML 透過自定義的 `<mj-style>` 標籤設定，即可自動將 CSS 套用到元素的 inline style 中。

```html
<mj-style inline="inline">
<!-- 自定義 CSS -->
</mj-style>
```

table 排版的部分，MJML 的元素們都會自動編譯成 Table 排版；例如最基本的 MJML 如下：

```html
<mj-section>
  <mj-column>
    <mj-text>
      Hello World!
    </mj-text>
  </mj-column>
</mj-section>
```

在編譯過後會變成這樣：

```html
<!-- 礙於篇幅，以下已省略判斷是否為 IE 的部分 -->
<div style="margin:0px auto;max-width:600px;">
  <table align="center" border="0" cellpadding="0" cellspacing="0" role="presentation" style="width:100%;">
    <tbody>
      <tr>
        <td style="direction:ltr;font-size:0px;padding:20px 0;text-align:center;">
          <div class="mj-column-per-100 outlook-group-fix"
            style="font-size:0px;text-align:left;direction:ltr;display:inline-block;vertical-align:top;width:100%;">
            <table border="0" cellpadding="0" cellspacing="0" role="presentation" style="vertical-align:top;"
              width="100%">
              <tr>
                <td align="left" style="font-size:0px;padding:10px 25px;word-break:break-word;">
                  <div style="font-family:Ubuntu, Helvetica, Arial, sans-serif;font-size:13px;line-height:1;text-align:left;color:#000000;">
                    Hello World!
                  </div>
                </td>
              </tr>
            </table>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

自動生成了支援各 Email Client 的巢狀 Table 排版

關於 RWD 的部分，MJML 可以透過 `<mj-section>`、`<mj-column>` 兩個標籤的搭配，快速堆砌出 RWD Email；`<mj-section>` 即代表一組內容，而 `<mj-column>` 預設即會自動均分寬度，在指定寬度後，便能自動適應寬度，流動排版，達到 RWD 的效果。 

最重要的特性，MJML 是以 Component Base 思維設計的，開發者可以撰寫組件，再透過 `<mj-include>` 引入，組件便能重複使用，多封 Email 維持一致的設計風格也就變成非常簡單的事情；MJML 本身也提供不少已經做好效果的組件，例如收合效果的 `<mj-accordion>`，大圖背景的 `<mj-hero>` 等等，參考 [官網文件](https://mjml.io/documentation/) 的說明，便能快速套用到專案中。

綜合前述種種功能，開發者得以從繁複的相容設定解放出來，只需要專注開發信件內容的呈現即可。

> 再提醒一下，在使用 MJML 時，Email Client 不支援的 HTML 標籤及 CSS 屬性，一樣都還是不支援喔！開發者撰寫時，還是需自行確認各屬性是否能被正確顯示；另外，對於 `<mj-raw>` 標籤內的自訂元素，MJML 也不會做任何處理，實作 RWD 排版時需要特別留心喔。

## 應用範例

如同本文最一開始提到的，這次會開始研究 RWD Email 的主因，還是因為工作上的需求，我需要實作出給各產品內部使用的 Email Template Service，因此我選擇 npm 套件版本的 MJML，並撰寫簡單的 node.js 後端，做出 API 供其他產品呼叫。

### 伺服器程式

主要是透過 [Express](https://expressjs.com/zh-tw/) 做伺服器基礎，搭配 [backpack](https://github.com/jaredpalmer/backpack) 做 bundle 的設定，基本的 logger、authenticator、api routers 都可以透過 middleware 實作，其他就沒有什麼特別的了。

要注意的地方是 `.mjml` 的副檔名預設不會被 watcher 捕捉到，自然也不會觸發程式 rebuild，開發者需要另外撰寫 backpack.config 做設定：

```js
const WatchExternalFilesPlugin = require('webpack-watch-files-plugin').default

module.exports = {
  webpack: (config, options, webpack) => {
    config.entry.main = ['./src/app.js']
    config.watchOptions = { 
      poll: true, aggregateTimeout: 300
    }
    config.plugins.push(
      new WatchExternalFilesPlugin({
        files: ['./src/**/*']
      })
    )

    return config
  }
}
```

### MJML + Handlebars

由於 MJML 僅為標記式語言，不提供帶入資料參數的方法，也無法做任何邏輯判斷，如果想要做到隨資料改變內容，就必須透過其他的模板引擎輔助。這部分我選用老牌的 [Handlebars](https://handlebarsjs.com/) 作為模板引擎。

> 搜尋資料時有看到其他開發者 [透過 Vue-Server-Renderer + MJML 實作信件模板](https://github.com/JamesClow/Vue-MJML)；由於 Vue 的核心功能（雙向綁定、事件驅動...）在這樣的場景完全無法使用，變成純粹只為了伺服器渲染而用 Vue，我認為這樣是多繞一圈的做法；但如果就是想用 Vue 的語法做開發，也是一種可以考慮的實作方式。

由於套用了 Handlebars，在模板中就可以透過 `{{ }}` 標記出資料區塊，以及各種 Handlebars helper 做邏輯判斷，與 MJML 搭配起來，一個能依據資料改變內容的 Email 模板就大功告成囉。

核心的處理邏輯會是：
1. 讀取、解析模板
2. 透過 handlebars 處理邏輯、填入資料
3. 透過 mjml 將處理後的模板轉譯成 html

程式碼範例如下：
```js
import { compile } from 'handlebars'
import mjml2html from 'mjml'

// 自訂了取得模板的 helper function
// 會回傳 fs.readFile 的結果
import { getMjmlTemplate } from '../utils/mjmlHelper'

export const htmlGenerator = (filename, data) => {
  try {
    // step 1. 取得 .mjml file（純文字）
    const mjmlTemplate = getMjmlTemplate(filename)
    
    // step 2. 將純文字解析成 handlebars 模板
    const hbsTemplate = compile(mjmlTemplate)

    // step 3. 將資料及邏輯處理塞入模板，解析成純 mjml
    const mjml = hbsTemplate(data)

    // step 4. 將 mjml 編譯成 HTML
    const html = mjml2html(mjml)

    // step 5. 取得編譯完成的 HTML
    return html
  } catch (e) {
    throw e
  }
}
```

### 前端程式

為了讓使用者能提前了解寄出的 Email 內容，專案內也實作了簡易的前台，透過 [JSON Editor](https://github.com/josdejong/jsoneditor) 呈現資料，及當前 Email 畫面的雙欄顯示，讓使用者能直接編輯資料、預覽信件內容：

![email preview](https://i.imgur.com/i2efomk.png)

由於 MJML 提供了 [json 格式轉 MJML 的方法](https://mjml.io/documentation/#using-mjml-in-json)，後續也許能進一步的實作出拖拉介面，讓 Email 模板的建立、編輯由需要的使用者自定，不需再經由工程師來切版撰寫模板。

## 結語

透過 MJML，開發者得以付出更少的時間，做出更能靈活呈現的 RWD Email；再搭配上 Handlebars 作為模板引擎，就能得到更加彈性的動態模板了。

以上大致紀錄了我與 Table 排版的對抗過程，稍微提及 MJML 的特色，並提供應用實例，希望能幫助到同為 Email 排版所苦的讀者們。如果文中有任何錯誤或不清楚的地方，都歡迎你提出一起教學相長；若你覺得本文有幫助到你，也請不要吝嗇你的掌聲～

https://button.like.co/gary_chu

### 參考資料

- [從頭開始構建一個 HTML Email 模板](https://webdesign.tutsplus.com/zh-hant/articles/build-an-html-email-template-from-scratch--webdesign-12770)
- [MJML](https://github.com/mjmlio/mjml)
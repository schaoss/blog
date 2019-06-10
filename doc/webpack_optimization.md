# 網站效能優化實錄 

![架構](https://i.imgur.com/XWCnHDv.png)

這篇文章主是想把近幾個月與團隊一起調教參數、設定，逐步優化專案的過程紀錄下來。

本文中提到的專案是透過 Nuxt 建置，因此本文主要會圍繞在 Nuxt、Webpack 的各項參數設定。

## 緣起

在 [Chrome 59 的更新內容](https://developers.google.com/web/updates/2017/04/devtools-release-notes?hl=zh-tw) 中，新增了程式使用率面版；當我注意到這個更新後，第一時間便對目前進行中的專案打開面版查看：

![before](https://i.imgur.com/96wN5sH.png)
> 用 Git 回到當時版本的模擬圖片

如上圖，使用率竟然只有驚人的 17 %；另外第一屏的載入時間也非常長，平均大約需要 3.2 秒。這些數據讓當時的我感到非常訝異，也就因此決定要利用空檔好好改善載入速度。於是就開始 Survey 該如何進行優化。

## 專案優化

### 程式碼切分

第一時間想到的是從 Webpack 的設定下手；Webpack 4 增加了 optimization 的設定，包含了許多面向的優化選項，而我就從切分程式碼的部分開始下手。

從先前的使用率面板，可以看到第一項是一整大包的 js 檔，但使用率只有不到 10%，這是因為先前沒有設定要進行程式碼切分，而程式在經過 Webpack 編譯時，便將 node_modules 及大部分的程式碼都放在同一隻 js 中。缺點非常明顯，實際上可能 A 套件只在特定頁面中使用，其他頁面卻需要花時間 & 流量下載它，便造成了浪費。

要改善其實很簡單，只要在 Webpack 4 中的 optimization 加入 [splitChunks](https://webpack.js.org/plugins/split-chunks-plugin/#splitchunksmaxinitialrequests)，並給定基本的參數即可：

```javascript
// 將 webpack 的 runtime 獨立成一個 chunk
runtimeChunk: true, 
// 切分 node_modules 中的套件
splitChunks: {
  chunks: 'all',
  minSize: 30000,
  maxSize: 0,
  minChunks: 1,
},
```

預設的設定中，Webpack 會依照各檔案之間的引用關係，把會被一起重複使用的部分拆分出獨立的 chunk，同時控制單一檔案最多只會引入 3 個 chunk，避免瀏覽器 request 數量被 chunk 被占滿。

除了 Webpack 的設定外，在 Vue Router 中也可以透過非同步載入的語法，讓 Webpack 將各頁面的內容單獨切分：

```javascript
...
routes: [
  {
    path: '/path1',
    name: 'page1',
    component: () => import('~/pages/Page1').then(c => c.default)
  },
  {
    path: '/path1',
    name: 'page2',
    component: () => import('~/pages/Page2').then(c => c.default)
  },
  ...
]
```


來看看這部分的結果吧：

![Webpack Bundle Analyzer](https://i.imgur.com/UuYfvbq.png)
> Webpack Bundle Analyzer

上圖是透過 [Webpack Bundle Analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 把檔案大小視覺化出來的結果，可以看的出來經過 Webpack 打包後的程式，已經被自動分組成幾隻不同的檔案了。左上角很大一塊的橘色方塊是網頁 3D 需要用到的相關套件，淺綠色則是 Nuxt 的相關模組，中邊淺灰色的部分是 Vue 的相關模組，右邊一堆小小的淺藍色方塊則是設定成非同步載入的各個頁面。

這樣就能初步讓頁面避免載入無用的程式了！

### Javascript Uglify

減少了無用的部分，再來是減少載入的大小；Webpack optimization 中可以設定 minimizer，這部份就交給 [UglifyJS Webpack Plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin) 來處理吧：

```javascript
minimize: !isDev,
minimizer: [
  new UglifyJsPlugin({
    cache: true,
    parallel: true,
    uglifyOptions: {
      output: {
        comments: false
      }
    }
  })
],
```

設定內容很簡單，使用快取、平行處理、移除註解，就不多做說明了。

### HTML & CSS Minify

Javascript 最小化了，再來是 HTML & CSS 的最小化。
[Nuxt 中已經整合了許多常用的Webpack plugin](https://zh.nuxtjs.org/api/configuration-build/)，只要簡單的加上設定即可；首先是 HTML：

```javascript
'html.minify': {
  collapseBooleanAttributes: true,
  decodeEntities: true,
  processConditionalComments: true,
  removeComments: true,
  removeEmptyAttributes: true,
  removeRedundantAttributes: true,
  trimCustomFragments: true,
  useShortDoctype: true
},
```

內容包含移除註解、移除無用的屬性、收合布林值的屬性等等，就是能省則省。

CSS 的部分，則是要透過 postcss 的設定。[cssnano](https://github.com/cssnano/cssnano) 是個常見的解決方案：

```javascript 
postcss: {
  plugins: {
    cssnano: {
      preset: [
        'default',
        {
          discardComments: {
            removeAll: true
          }
        }
      ]
    }
  },
  preset: {
    autoprefixer: {
      browsers: ['> 1%', 'last 3 versions']
    }
  },
  order: 'cssnanoLast'
},
```

> 因為還有設定 auto-prefixer，必須要額外指定 order: 'cssnanoLast'，讓 cssnano 最後執行。

當時專案內使用了 Element-UI，並有整包引入再客製化內容，CSS 佔據專案大小占比蠻大一塊的；在 CSS minify 設定完成後，編譯出的檔案大小馬上有可觀的下降。

### 更換套件

做到這邊，已經把基本能節省的地方都處理完了；如果還想要更進一步的優化，就必須往移除 / 更換套件的方向走了。

第一個目標是知名的時間處理套件 - [Moment.js](https://momentjs.com/)；先前版本的 Moment.js 預設會載入各地的 Locale 檔案，一整組的大小來到 200+ kB，相當的龐大；因此我們選擇更換到號稱只有 2kB 的 [Day.js](https://github.com/iamkun/dayjs)。

![Day.js](https://user-images.githubusercontent.com/17680888/39081119-3057bbe2-456e-11e8-862c-646133ad4b43.png)

在 Day.js 有意的設計下，兩者的 API 設計幾乎相同，由於專案內沒有太複雜的時間計算，幾乎算是無痛更換了。

這樣一換好，又省去了 200+ kB

### 後端

前端優化到一個段落，載入時間大概降低了一秒半，接下來就交棒給後端啦。

經過一陣子的討論 & 研究，我們發現：由於 Nuxt 是 SSR 框架，當它要取得第一畫面的資料，向後端 API 要資料時，會先向 Route 53 取得後端伺服器的外網 IP 位置，再送出 request；但其實這一段是可以走內網的。

request 繞出去再繞回來，一來一往也浪費了不少時間，透過設定好內網路徑 domain，第一畫面的回應時間又再次大幅下降。

### 其他

除了前述比較工程的部分外，也有把一些外部的 css 直接引入專案中，省去一次來回的 request，也讓第一畫面的樣式能直接正確顯示。

其他還有一些優化的方向就比較與程式無關了，主要是把過大的圖檔素材做壓縮優化；除了程式的部分外，圖檔的大小對載入時間也會影響非常大的一部分。

## 優化結果

整理了前後對比如下：

### 第一屏回應時間
![第一屏回應時間](https://lh6.googleusercontent.com/Uf5xslU2IhAOIEUC6mpCAKw3a8vJSUx4Kyd9YpnD0ocOXoN78jDLtfDr0Z6moU-ZRd-EfN51xPxpti3JkzCyrbBlUXjuFdodS3CCNHkW)

### 下載內容總量
![下載內容總量](https://lh6.googleusercontent.com/XSqkjLGhFVQjf3HbsCuxnpWZ4LLLYfnON48IQPrPrHWPK5AxzRUtcYWYlS4jkp57IA2c3yz46etZcmPPiaSyEhLhbEviYJXzbLWdBJBS)

### 下載內容無用率
![下載內容無用率](https://lh3.googleusercontent.com/2d8kXw_e4iq0dBn9Hca6oojutOSfwdOazX94avvJPlBC9DriJfHb1dxpSrODmuZDhaEChJJx4RQuQkXEMYbqbtiU4UwRLM9IR1hc19W9)

> 首頁： Landing page
> 活動列表頁：後端 API 回應資料組出列表
> 虛擬展覽頁：後端 API 回應資料 & 3D 互動

整體來說，算是非常有感的優化吧；第一屏的回應時間平均就減少了 2.5 秒，下載內容總量也很符合預期，主要是讓一般頁面不用再下載網頁 3D 龐大的套件。

附上一個 Google Audits 的結果：
![Google Audits](https://lh4.googleusercontent.com/1tyAeiuvb-3Sw86HvYqZRHkhFGq9HuzF85E5Ed3312eSBB2aO5rAK59n4wRD5p2Uk1e_C615SbaRTtP8aOBbPp-y_QV8HLINvk6bYazew4GxPT0mu3srfMMh1kB4PKb1Ia-sm0_Sgrs)

## 結論

江湖傳言，[工程師有三大美德：懶惰、急躁、傲慢](https://zh.wikipedia.org/wiki/%E6%8B%89%E9%87%8C%C2%B7%E6%B2%83%E5%B0%94)，身為一個網站工程師，瀏覽到效能表現貧弱的網站總是令我感到非常急躁，當發生在自己產品上時就更是如此了；這次的優化經驗中，由畫面到路由設定，跨越了蠻大的範疇，過程中也有許多眉眉角角，所幸最後的結果比預期還好，真的是一次很棒的經驗累積！

那麼這篇實錄就寫到這啦，如果讀者您有相關的優化心得，都歡迎您在下面一起回應討論；若文中有任何不清楚或錯誤的地方，也請您不吝告知。
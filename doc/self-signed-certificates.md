# 開發用 SSL 自簽設定流程

由於最近在開發與第三方登入有關的服務，需要設定 SSL 自簽才能正確與第三方服務溝通。本文把設定的過程一步一步的紀錄下來，帶你手把手的完成 SSL 自簽設定。

## 什麼是 SSL 自簽

**SSL** 為 _Secure Sockets Layer (安全通訊協定)_ 的縮寫，後來發展成 Transport Layer Security，[傳輸層安全協定 (TLS)](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)；簡單說就是在通訊的過程中，透過特定的演算法把傳遞內容加密，防止內容被竊取或竄改。例如我們常見的 [https](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE) 即為透過 SSL/TLS 加密 http 傳送的內容。

**自簽** 即為自己簽屬的 [憑證](https://zh.wikipedia.org/wiki/%E6%86%91%E8%AD%89)。
**憑證** 是用來證明 **「這個網址是誰」**，是用來設定 SSL 的。

> 準確的說應該為 「這把公鑰是誰的」，不過先這樣理解應該比較好懂 XD。

例如，為什麼我們可以知道 `https://www.google.com/` 是 Google 而不是 hao123？因為瀏覽器信任了一連串的憑證，證明了這個位置就是 Google。

![憑證](https://i.imgur.com/HCCC3pb.png)

## 為什麼需要 SSL 自簽

由於 https 使用[非對稱性加密](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)，仰賴 [公開金鑰認證](https://zh.wikipedia.org/wiki/%E5%85%AC%E9%96%8B%E9%87%91%E9%91%B0%E8%AA%8D%E8%AD%89)，因此在本地開發時，你的測試環境就會一直被瀏覽器及部分安全性套件判定成 _不安全_；設定 SSL 自簽，即為 **透過一組自己發行、自行認證、自己相信的憑證，來設定 SSL 加密**，就可以騙過瀏覽器及安全性套件，讓你順利的完成開發測試。

也就是說，這是一個 自己騙自己 的過程。

## 準備 mkcert、Nginx

需要先準備這兩個工具：

1. [mkcert](https://github.com/FiloSottile/mkcert)，用來生成 _自簽憑證_。
2. [Nginx](https://nginx.org/en/download.html)，作為 _反向代理伺服器_。

Mac 也可以透過 `brew` 直接安裝

```bash
$ brew install mkcert
$ brew install nginx
```

如果憑證或反向代理伺服器有別的替代方案，也可以自行設定、準備。

## 生成憑證

```bash
$ mkcert -install

$ mkcert your-domain-name.com "*.star.104.com.tw" localhost 127.0.0.1 ::1
```

> 這邊請設定你想要設定到的 domain，不要傻傻的複製貼上啊結果不能用啊 XD

正確完成後，你會得到兩隻檔案：

- certificate： \*.pem
- private-key： \*-key.pem

## 設定反向代理

在 `nginx.conf` 的 server 區塊中，加上剛剛生成的 key。
例如：

```nginx
server {
  server_name accounts.star.104-local.com.tw;
  listen  80;
  listen  443   ssl;
  ssl_certificate      D:\104Crop\key\_wildcard.star.104-local.com.tw.pem;
  ssl_certificate_key  D:\104Crop\key\_wildcard.star.104-local.com.tw-key.pem;
  location / {
    proxy_pass http://127.0.0.1:3333;
  }
}
```

> 上例中 `server_name` 用了 `hosts` 檔案中額外設定的別名，一樣是指到 `127.0.0.1:3333`。

以上設定是讓 Nginx 去聽 443 port 提供 SSL (https) 的服務。並將 req 轉傳給原本 http 的服務，也就是反向代理。

## 驗證結果

由 command-line 啟動 Nginx：

```bash
$ nginx
```

接著便能打開瀏覽器，透過 `https://your-domain.com` 之類的，確認是不是有設定成功。

如果 `nginx.conf` 設定錯誤，在修改後需要將 Nginx 關閉再重啟，才能看到效果；
Mac 可以透過：

```bash
$ nginx -s stop
```

來關閉 Nginx，Windows 則只能從 _工作管理員_ 砍掉 process 囉。

> 嫌麻煩可以註冊成 Windows 的服務，透過服務開關。

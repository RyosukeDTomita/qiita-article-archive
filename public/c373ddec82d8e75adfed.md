---
title: 特定のサイトで日本語フォントがおかしいときの対処法 chatgpt.comの「、」「。」が文字化けした
tags:
  - Firefox
  - font
  - error
  - ChatGPT
private: false
updated_at: '2026-02-13T15:09:29+09:00'
id: c373ddec82d8e75adfed
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

一昨日くらいからchatgpt.comの日本語フォントがおかしくなった。
具体的には「、」と「。」の形がおかしい。


![Screenshot from 2026-02-13 14-25-44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/1b0e51c6-ce79-48db-b51d-bb473a8a9db5.png)


これを修正する方法を書く。

---

## 環境

- Ubuntu 24.04 LTS

  ```shell
  cat /etc/os-release
  PRETTY_NAME="Ubuntu 24.04.3 LTS"
  NAME="Ubuntu"
  VERSION_ID="24.04"
  VERSION="24.04.3 LTS (Noble Numbat)"
  VERSION_CODENAME=noble
  ```
- Firefox 147.0.3 (64-bit)
- 対象サイト: chatgpt.com
  ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/6df58f66-36aa-41f0-8cfc-1ddf4915e77d.png)
- システム言語: English (United States)
  ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/6d4dc19d-0b32-4146-a117-3441e2d938af.png)


---

## 解決方法: CSSに日本語フォントを追加した

https://addons.mozilla.org/ja/firefox/addon/styl-us/

```css
* {
    font-family: "Noto Sans JP", "Hiragino Sans", "Yu Gothic",
      system-ui, -apple-system, "Segoe UI", sans-serif !important;
  }
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/964ef04d-2bb1-4886-9a48-cc857ce59cb1.png)

---

## 試したこと

- UbuntuのシステムfontをNoto Sans CJK JPに変更 --> 変化なし
  ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/a4ec405d-1c73-4ad4-b955-44544fa6c330.png)
- Chromeでアクセスしてみる --> 異常なし。
- 他サイトを試す --> 異常なし
- FirefoxのフォントをNoto Sans CJK JPに変更 --> 変化なし
- chatgpt.comの言語設定を日本語にする --> 文字化け継続
- 開発者ツールからconsoleで以下を入力 --> 直った! --> Stylusでcss追加した。
   ```js
   document.body.style.fontFamily = '"Noto Sans JP", sans-serif'
   ```
   ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/53691ccf-7d37-40f2-b361-f600f7be4819.png)

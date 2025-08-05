---
title: MarpとGensparkを使ってきれいなスライドを作る方法 + SpeackerDeckアップロード用にhtmlをpdfに変換する
tags:
  - SpeakerDeck
  - LT
  - puppeteer
  - 発表資料
  - Genspark
private: false
updated_at: '2025-07-30T00:47:09+09:00'
id: a3fdcf72bce350d3f8c7
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

今回始めて[Gensparkのスライド機能](https://www.genspark.ai/agents?type=slides_agent)を試したのですが、画像が壊れたりしたのでローカルで修正したくなりました。

その後pdf化して[SpeakerDeck](https://speakerdeck.com/)に上げるまでに多少ハマったポイントを解決した方法を記載します。

---

## まず、Marpよりはじめよ

いきなりGensparkに依頼しても思っているようなスライドはできないと思っています。

そのため、私は以下のフローで進めました。

1. Qiitaを書く。 [実際に書いたQiita記事](https://qiita.com/sigma_devsecops/items/a9e91ea580b16bc90ee5)
2. AIにQiitaをもとにMarpを作ってもらう
3. Marpを編集する。

- Marpを挟むことで編集しやすい媒体であるMarkdownでスライドの構成を考えることができること
- GitHubで資料を管理できること
- Gensparkでいい感じの資料が作れなくても最悪Marpで発表できる

というメリットがあると思います。

---

## Marpで作った資料をGensparkに投げる

これを元に資料を作ってでおまかせしました。

ほとんど内容が変わらずにデザイン性が向上しました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/f47dc657-5c70-48cd-beee-dd7fe23aea88.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/e909e4a9-2ef3-4d75-b7e6-09610312d1f6.png)

ただ、自分が試した感じだと画像の取り間違い、透明背景の画像が黒背景になってしまう等、画像周りがうまくいきませんでした。

---

## Gensparkの資料をhtmlでexportできなかったのでコピペで頑張る

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/9769cec7-f99b-4fe4-81d5-9f3a2d869b27.png)

自分がさっと見た感じだとpptxとpdf exportしかサポートしていなさそうでした。

そのため、スライド1枚ずつコピペでhtmlを保存し、1.html、2.htmlという名前をつけてローカルに保存し、それをClaude Codeに結合してもらいました。

これで細かい修正等をClaude Codeにやってもらえるようになりました。

---

## SpeakerDeckに載せるためにpdf化する

ブラウザの印刷機能でexportしようとするとファイル名や日付が入ってしまいます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/e8e8e454-c013-49aa-9a10-9b282e5dd2d9.png)

(ブラウザの設定いじればどうにかできそうな気はしつつも)、[Puppeteer](https://pptr.dev/)を使ってhtml pdf変換を行いました。

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('file:///home/sigma/marp_env/20250730_qiitabash/tdd.html', { waitUntil: 'networkidle0' });
  await page.pdf({ 
    path: 'tdd.pdf', 
    width: '14.54in',
    height: '8.18in',
    printBackground: true,
    margin: {
      top: '0px',
      right: '0px',
      bottom: '0px',
      left: '0px'
    }
  });
  await browser.close();
})();
```

スライドは横長なので、直接`width`、`height`を指定しています

```shell
npm i puppeteer
node html_to_pdf.js
```

---

## 終わりに

- pdf→html変換がおもったよりもうまくいかなかった
  - つべこべいわずに、Gensparkに課金するほうが早そう
- Claude Code単品でhtmlできれいなスライドを作れないか気になる

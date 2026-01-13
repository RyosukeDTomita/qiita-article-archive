---
title: Ubuntu 22.04日本語RemixでUSキーボードになってしまったときに日本語キーボードに戻す方法
tags:
  - error
  - USキーボード
  - Ubuntu22.04
  - 日本語キーボード
private: false
updated_at: '2026-01-12T12:13:22+09:00'
id: 0cec4dce31a997fdff0c
organization_url_name: null
slide: false
ignorePublish: false
---
## 発生した事象

再起動したら使っているキーボードが日本語キーボードとして認識されず、USキーボードになってしまった。

---

## 環境

- Ubuntu 22.04(日本語Remix)
- Japanese (Mozc)

https://qiita.com/sigma_devsecops/items/c046e7ba1a19bb21e9ed

---

## 原因と解決策

時々、Japanese (Mozc)が落ちてしまい、日本語入力ができなくなる事象が発生していたので、設定のInput Sourcesにあったjapaneseを消したのが原因だった。

設定からInput Sourcesに「Japanese」を追加したら日本語キーボードに戻った。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/dc6b1efe-1095-460e-8177-6c3d8b0acd2e.png)

日本語Remixを使っていてもともと入っていたものは消さないほうが良いかもしれない。

---
title: 「Clineに全部賭ける」勇気がでないのでGitHub Copilot Agentでお安く試してみる
tags:
  - GitHub
  - githubcopilot
  - 生成AI
  - AIAgent
  - cline
private: false
updated_at: '2025-02-27T02:38:55+09:00'
id: cd420bd54cbbe1c40cc0
organization_url_name: nri
slide: false
ignorePublish: false
---
## はじめに

mizchiさんの記事を読んで以下の部分に衝撃を受けた。

> ペアプロでいうとClineがドライバーで、ユーザーがナビゲータになる。役割を交代する。

https://zenn.dev/mizchi/articles/all-in-on-cline

ここ2ヶ月ほどの間自分は生成AIの作ったコードを評価するために

- 設計，デザインパターン
- リファクタリング
- TDD，テストとの向き合い方

を主に学習していたが，これは自分がAIエージェントにサポートを受けることを前提に考えてのことだった。

今後，我々人間がAIのサポートをするような時代がくることを考えると，早いうちにAIエージェントとうまく付き合っていくやり方を考えるのは急務であると思われ，そのためにはAIエージェントをできるだけ自分で使ってみるべきであると思われる。

しかし，Clineをゴリゴリつかうと1時間で5ドルが飛んでいくくらしく，貧乏性な上に個人事業主でもない自分にはコストが高すぎる。

前置きが長くなってしまったが，とりあえずお安くAIエージェントと遊ぶために[GitHub Copilot AGENT MODE](https://github.blog/jp/2025-02-07-github-copilot-the-agent-awakens/)を使った際の防備録としてこの記事を書いた。

---

## 環境構築方法

### VSCodeのInsiders版をインストール

2025年2月27日現在では，GitHub Copilot AGENT MODEはVSCodeのInsiders版でのみ提供されている。

[VSCode Insiders](https://code.visualstudio.com/insiders/)からインストールする。

### CopilotとCopilot Chatのプレリリース版をインストール

Insider版のVSCodeのインストールが済んだら，CopilotとCopilot Chatのプレリリース版をインストールする。
やることは至って簡単で，通常のExtensionsを入れるのと同じだ。

### AGENT MODEに切り替えてみる

AGENT MODEに切り替えるにはまず，GitHub Copilot Chatの画面から，Copilot Editに移動した後，切り替え可能になる。

1. Copilot Chatを開く
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/11c256e7-dc2a-423d-a9b2-a8d5764cabdb.png)
2. Copilot Editに変更
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/db7ecb90-9f65-4678-b4a7-89a5af958e79.png)
3. AGENT MODEに切り替え
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/a308f114-36ca-4aa3-af92-a7698c78030f.png)

---

## とりあえず，使ってみる

### 1: テストを書いてもらう

簡単なflask cliのコマンドに対する単体テストを書いてもらった。
特に許可なども求められず，新規ファイルが作成され，単体テストが作成された。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/bf0b848b-44bb-47ba-ab2f-8bd5dceb593b.png)

### 2: テストを実行してもらう

テストを実行するたびに許可が求められた。
承認されるまで動いてくれないというのも面倒なのでどうにかならないか調べてみた。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/9e36beb3-4df4-427d-91fc-393fc968cb71.png)

[Introducing GitHub Copilot agent mode (preview)](https://code.visualstudio.com/blogs/2025/02/24/introducing-copilot-agent-mode)によると，

> To easily intervene and undo in those situations, every tool invocation is transparently displayed in the UI, terminal tool requires approval

とあるので許可をオフにする設定は2025年2月27日現在はないようだ。

:::note info
無理やりお願いしてみたがだめだった。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/646e9dac-62c2-4406-b3e7-33efcb0fac84.png)
:::

### 3. TDDをやってみる

新機能を作る前にテストを作成し，このテストが通るようにしてとAIエージェントにお願いする作戦。
個人的にはTDDをうまく使えればAIエージェントの生産性をめちゃくちゃ向上させられるのではと思っている。

- タスクの終了条件が明確になる
- コードのアーキテクチャ等を人間がチェックした上でAIエージェントにタスクをお願いできる

というのがやはり強いのではないか。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/6d2f5a57-2662-48eb-b8b8-fd8c1e5c5b9d.png)

### 4. AIエージェントに対して普段から使うルールを登録しておく

Clineであれば，`.clinerules`に記載しておけばある程度制御が可能らしい。
GitHub Copilotの場合`.github/copilot-instructions.md`というのがこれにあたりそうなので試してみる。

自分は常にTDDで実装してもらえるように以下を追加してみた。

```
## 重要

実装時には必ずTDDで行ってください。

1. 依頼内容を1つの単体テストになるまで細分化する
2. 上記の単体テストを実装する
3. 単体テストをとりあえず，通すようにソースコードを書く
4. ユーザにレビューを依頼する
5. テストが通ることを確認しながら実装を進める
6. 機能ができあがったら，重複を削除するためにリファクタリングを行う
```
特に`.github/copilot-instructions.md`をアクティブなタブとして開いていないが，上記を考慮してタスクを実施してくれてそうではある。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/cba2c550-4b27-41dd-82d5-52d2b52e8b41.png)

:::note info
「語尾にござるをつけて」copilot-instructions.mdに書いてみたらちゃんとやってくれた
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/b7411607-d56f-499e-82e7-db6056424e5c.png)
:::

---

## まとめと感想

- Clineと違ってコマンドの承認が必要な点を考慮するとまだ，AIに任せる度合いは少なめではある。
- APIを叩くたびに課金されないので安心して使えるので，とりあえず試してAIエージェントに慣れるのは良さそう。
- copilot-instructions.mdを書くことである程度制御ができそうなので，これの書き方には研究の余地がありそう。

---

## 備考

- TDDをやる良い方法を「テスト駆動開発」の訳者のt_wadaさんとmizchiさんが話してる[このツリー](
https://x.com/mizchi/status/1893982462159544692)参考になりそう




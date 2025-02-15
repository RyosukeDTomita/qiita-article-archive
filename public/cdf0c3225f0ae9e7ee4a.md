---
title: Docker版「バルス」不要リソースをまとめて消してストレージの空き容量を回復する
tags:
  - Docker
  - Cache
  - container
  - rm
  - バルス
private: false
updated_at: '2024-09-02T19:51:08+09:00'
id: cdf0c3225f0ae9e7ee4a
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

Dockerを使っているとコンピュータの記憶領域のリソースが足りなくなることがあります。
また，無駄なコンテナやイメージをまとめて消したくなる時もあります。

この記事では，Dockerで作成したすべてのものをまとめて削除する方法を書きます。
実行は自己責任でお願いします。

この記事は前記事の続編のバルスシリーズです。

https://qiita.com/sigma_devsecops/items/c1ec056a80f12f151ed7

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/18949b1f-9a98-6d6a-2213-db24c3e7e547.png)
(笑)

---

## コマンド

### imageの全削除

```shell
docker image rm $(docker image ls -q) -f
```

```shell
docker rmi $(dokcer images -q) -f
```
`-q`オプションを使うとコンテナIDやImage名だけを返してくれるのでシェルスクリプトを扱う上でめちゃくちゃ便利です。

どちらのコマンドを使っても一緒ですが，古いバージョンのほうが自分は個人的には文字数が少ない後者が好きです。

### containerの全削除

```shell
docker container rm $(docker container ls -q -a) -f
```

```shell
docker rm $(docker ps -q -a) -f
```
これもどちらを使っても同じです。`-a`オプションで起動していないコンテナも削除しているのがミソです。

### キャッシュの全削除

普通に数十GB空きスペースが増えたりするので定期的に実行すると良いです。

```shell
docker builder prune
```

---

## 終わりに

それではみなさんも快適なdocker版「バルス」ライフを!


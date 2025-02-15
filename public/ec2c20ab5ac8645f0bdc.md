---
title: 【SIer必見】AsciiDocとPlantUMLでドキュメント作成環境を作った
tags:
  - document
  - gradle
  - PlantUML
  - asciidoc
  - SIer
private: false
updated_at: '2024-11-07T19:12:13+09:00'
id: ec2c20ab5ac8645f0bdc
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

Asciidoctor（AsciiDoc）は、特に技術文書やプロジェクトドキュメントの作成でよく使用されるマークアップ言語です。Markdownよりも柔軟な表現ができる一方で，html等よりも記述が簡単であり，htmlやpdfとしてexportすることもできます。
しかし，意外と環境構築が面倒です。

この記事ではDocker/DevContainerを使ってすぐにAsciiDocをbuildできる環境を作成しました。

---

## AsciiDocとは

- Asciidoctor（AsciiDoc）は、特に技術文書やプロジェクトドキュメントの作成でよく使用されるマークアップ言語の一つ
- markdownよりも表現力がある一方で，HTMLような高い表現力

自分の記事ではないですが，このQiitaがわかりやすかったので貼っておきます。

https://qiita.com/xmeta/items/de667a8b8a0f982e123a#-%E3%81%AA%E3%81%9Casciidoc%E3%82%92%E4%BD%BF%E3%81%86%E3%81%AE%E3%81%8B

---

## ビルド方法の歴史

もともとは[asciidoctor](https://github.com/asciidoctor/asciidoctor/blob/main/README-jp.adoc)というRuby製のツールを使ってビルドを行うのが一般的でした。
しかし，関連ツールの依存性を解決したりPlantUMLと連携することを考えるとコマンドがとても長くなってしまいます。
そこで，[Asciidoctor Gradle Plugin](https://asciidoctor.org/docs/asciidoctor-gradle-plugin/)が作成され，build環境を整えやすくなったという歴史的背景があるようです。
そのため，本記事でもGradleを使ってbuildを行います。

---

## AsciiDocをビルドしてみる

[今回作成したサンプルリポジトリ](https://github.com/RyosukeDTomita/asciidoc_env)ではDockerでBuild環境を整えました。詳しくはこちらをご覧ください。

::: note info
VSCodeのDev Containerについてはスペースの問題上割愛します。
Dev Containerについては

https://qiita.com/sigma_devsecops/items/79db1640d1a1cd82343d

を参考にしてください。
:::

```dockerfile
FROM gradle:8.10.2-jdk17-jammy AS devcontainer
WORKDIR /app

RUN apt-get update -y

RUN <<EOF bash -ex
apt-get install -y --no-install-recommends libxext6 libxrender1 libxtst6 libxi6
rm -rf /var/lib/lists
EOF

CMD ["gradle", "asciidoctor"]
```

```shell
git clone https://github.com/RyosukeDTomita/asciidoc_env
cd asciidoc_env
docker buildx bake
docker compose up # ./build配下にhtmlが作成される。
```

前述したようにビルドにGradleを使うのでGradleイメージをベースイメージに使用しています。
また，PlantUML作って作成したシーケンス図等を埋め込むため，必要なツールをインストールしています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/eaf4c63c-b0e6-247d-adfd-3da639d6d5c0.png)

最低限の構成だとこんな感じになります。テーマ等もカスタマイズ可能なので，Orilly用のテーマとか探してみても面白いかも。

---

## 追記

日本語が重なる場合には日本語フォントをいれると解決できた。

https://qiita.com/yomon8/items/a3e016b7ffc3e4fbb7e5

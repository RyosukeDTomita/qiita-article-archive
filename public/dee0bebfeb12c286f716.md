---
title: >-
  javax.net.ssl.SSLHandshakeException: PKIX path building failed:
  VPN環境下でJavaのビルドが失敗する原因と解決方法
tags:
  - Java
  - error
  - VPN
  - gradle
private: false
updated_at: '2026-03-24T09:28:22+09:00'
id: dee0bebfeb12c286f716
organization_url_name: null
slide: false
ignorePublish: false
---
## 起きている事象

会社で[自分が趣味で開発したツール](https://github.com/RyosukeDTomita/odin)を配布しようと思い、VPN環境下でビルドすると証明書検証に失敗する事象が発生した。

```
./gradlew shadowJar
Downloading https://services.gradle.org/distributions/gradle-8.7-bin.zip
javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

### 環境

- ホストOS: Windows 11 Pro
- WSL2 in Ubuntu 22.04 LTS

---

## 解決方法

VPNクライアントがTLS Inspectionに使用しているルート証明書をJavaのcacertsにimportすれば直る。

### ルート証明書を見つけてExportする

1. Win + r --> certlm.msc
2. 信頼されたルート証明書 --> 証明書 からVPNのルート証明書を探してexportする

### exportした証明書ををJavaのcacertsにimportする

:::note info
自分は[Nix Flake](https://wiki.nixos.org/wiki/Flakes/ja)を使っているため、以下のコマンドでうまくいったが、ビルドに使用しているJavaのcacertsにimportする必要があるので適宜書き換えること。
:::

```
sudo keytool -import -trustcacerts \
  -alias <お好きなエイリアス名> \
  -file <exportした.cer> \
  -keystore "$(dirname $(readlink -f $(which java)))/../lib/security/cacerts"
```

これで再度ビルドしたら成功した。

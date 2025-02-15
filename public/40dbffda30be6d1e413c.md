---
title: AWS LambdaをServerlessFrameworkでデプロイして最新のCVE情報を知りたい
tags:
  - cve
  - AWSLambda
  - ServerlessFramework
  - NIST
private: false
updated_at: '2024-12-19T22:11:23+09:00'
id: 40dbffda30be6d1e413c
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

セキュリティエンジニアたるもの，最新の脆弱性のニュースぐらい拾っておきたいなと思ったので
AWS Lambdaを使ってCVEが載っているサイトから最新の脆弱性情報を取得してみました。

[My GitHub Repository](https://github.com/RyosukeDTomita/cve_checker)

---

## 実装方針

AWS Lambdaで日時でスクリプトを実行する
- [ServerlessFramework](https://www.serverless.com/framework/docs/getting-started)でLambdaにデプロイする
- コードはPythonで書くスクレイピングして最新のCVEとそのURLを取得する
- Lambdaの実行結果を送信する(Slack，X，メール)?

---

## 環境構築

### Serverless

[公式のGetting Started](https://www.serverless.com/framework/docs/getting-started)の通りにセットアップ

- serverless install

```shell
npm i serverless -g
```

- `serverless`を実行
    - Templateは選択: 用途的にAPIとかではなさそうなので`AWS / Python / Scheduled Task`
    - Project名を設定
    - Serverless未登録なら`Login/Register`を選択するとブラウザが起動して初回登録を行う
    - IAM Role作成: AWSログイン後にターミナルに表示されたURLに飛ぶと初期設定用のCloud Formationのスタックを作成するように言われるので実行

- deployしてみる

```shell
cd <プロジェクト名>
serverless deploy
```
:::note info
`serverless`は`sls`と省略できる
(本記事ではわかりやすさのため省略記法は使わないが)
:::

サンプル用のLambdaがデプロイでき，実行できることをブラウザから確認。

:::note alart
初期テンプレートだと1分に1回Lambdaが実行されるようになっているので変更した方が良さそう
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/bc2b1a5d-a55a-2b01-0995-23f57672a25f.png)

```serverless.yml
      - schedule: rate(1 day) # 変更
```

(GUIだとAmazon Event BridgeのBuses→Rules→Event Schedulesから変更可能)
:::


---

## CVEの一覧を取得する

NISTのページから拾ってくることにしました。

- [API-KEY取得ページ](https://nvd.nist.gov/developers/request-an-api-key)
- [API Referene](https://nvd.nist.gov/developers/vulnerabilities)

雑にPythonのコードを書いて必要な情報だけパースしてローカルでの動作確認まで完了。

```python3
# coding: utf-8
"""_summary_
CVE情報を取得するLambda関数

Returns:
    _type_: _description_
"""
import requests
from datetime import datetime, timedelta, timezone
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)


def run(event, context):

    # NVD REST API URL
    NVD_API_URL = "https://services.nvd.nist.gov/rest/json/cves/2.0"
    API_KEY = "your api key"

    # CloudWatch Logsにログを出力
    current_time = datetime.now().time()
    logger.info("Your cron function ran at " + str(current_time))

    print("Fetching recent CVEs from NVD...")
    cve_items = fetch_recent_cves(url=NVD_API_URL, api_key=API_KEY, days=7)  # 過去7日間のCVEを取得

    if cve_items:
        print(f"Retrieved {len(cve_items)} recent CVEs:")
        display_cves(cve_items)
    else:
        print("No recent CVEs found.")


def fetch_recent_cves(url: str, api_key: str, days: int) -> list:
    """
    最近追加されたCVEを取得
    :param days: 過去何日分のCVEを取得するか (int)
    :return: 取得したCVEリスト (list)
    """
    end_date = datetime.now(timezone.utc)
    start_date = end_date - timedelta(days=days)
    all_cves = []
    start_index = 0
    results_per_page = 10

    params = {
        "resultsPerPage": results_per_page,
        "startIndex": start_index,
        "pubStartDate": start_date.strftime("%Y-%m-%dT%H:%M:%S.%fZ"),
        "pubEndDate": end_date.strftime("%Y-%m-%dT%H:%M:%S.%fZ"),
    }
    headers = {
        "apiKey": api_key
    }
    try:
        response = requests.get(url, headers=headers, params=params)
        response.raise_for_status()
        data = response.json()
        cve_items = data.get("vulnerabilities", [])
        for item in cve_items:
            cve = item.get("cve", {})
            cve_id = cve.get("id", "N/A")
            published_date = cve.get("published", "N/A")
            vuln_status = cve.get("vulnStatus", "N/A")
            reference_uri = [ref.get("url", "N/A") for ref in cve.get("references", [])]
            description = cve.get("descriptions", [{}])[0].get("value", "N/A")
            impact_score = cve.get("metrics", {}).get("cvssMetricV31", [{}])[0].get("cvssData", {}).get("baseScore", "N/A")
            all_cves.append({
                "id": cve_id,
                "published": published_date,
                "vuln_status": vuln_status,
                "reference_uri": reference_uri,
                "impact_score": impact_score,
                "description": description
            })
        start_index += results_per_page
    except requests.RequestException as e:
        print(f"Error fetching CVEs: {e}")

    return all_cves


def display_cves(cve_items):
    """
    CVE情報を表示
    :param cve_items: CVE情報のリスト
    """
    for cve in cve_items:
        cve_id = cve["id"]
        published_date = cve["published"]
        vuln_status = cve["vuln_status"]
        reference_uri = cve["reference_uri"]
        description = cve["description"]
        impact_score = cve["impact_score"]

        print(f"CVE ID: {cve_id}")
        print(f"Description: {description}")
        print(f"Published Date: {published_date}")
        print(f"Vulnerability Status: {vuln_status}")
        print(f"reference uri: {reference_uri[0]}")
        print(f"Impact Score: {impact_score}")
        print("-" * 50)


if __name__ == "__main__":
    run(None, None)

```

```shell
python3 handler.py                                                                       
Fetching recent CVEs from NVD...
Retrieved 10 recent CVEs:
CVE ID: CVE-2024-10043
Description: An issue has been discovered in GitLab EE affecting all versions starting from 14.3 before 17.4.6, all versions starting from 17.5 before 17.5.4 all versions starting from 17.6 before 17.6.2, that allows group users to view confidential incident title through the Wiki History Diff feature, potentially leading to information disclosure.
Published Date: 2024-12-12T12:15:21.330
Vulnerability Status: Awaiting Analysis
reference uri: https://gitlab.com/gitlab-org/gitlab/-/issues/499577
Impact Score: 3.1
--------------------------------------------------
CVE ID: CVE-2024-11274
Description: An issue was discovered in GitLab CE/EE affecting all versions starting from 16.1 prior to 17.4.6, starting from 17.5 prior to 17.5.4, and starting from 17.6 prior to 17.6.2, injection of NEL headers in k8s proxy response could lead to session data exfiltration.
Published Date: 2024-12-12T12:15:22.267
Vulnerability Status: Awaiting Analysis
reference uri: https://gitlab.com/gitlab-org/gitlab/-/issues/504707
Impact Score: 8.7
--------------------------------------------------
CVE ID: CVE-2024-12292
Description: An issue was discovered in GitLab CE/EE affecting all versions starting from 11.0 prior to 17.4.6, starting from 17.5 prior to 17.5.4, and starting from 17.6 prior to 17.6.2, where sensitive information passed in GraphQL mutations may have been retained in GraphQL logs.
Published Date: 2024-12-12T12:15:22.470
Vulnerability Status: Awaiting Analysis
reference uri: https://gitlab.com/gitlab-org/gitlab/-/issues/475211
Impact Score: 4.0
--------------------------------------------------
CVE ID: CVE-2024-12570
Description: An issue has been discovered in GitLab CE/EE affecting all versions starting from 13.7 prior to 17.4.6, from 17.5 prior to 17.5.4, and from 17.6 prior to 17.6.2. It may have been possible for an attacker with a victim's `CI_JOB_TOKEN` to obtain a GitLab session token belonging to the victim.
Published Date: 2024-12-12T12:15:22.660
Vulnerability Status: Awaiting Analysis
reference uri: https://gitlab.com/gitlab-org/gitlab/-/issues/494694
Impact Score: 6.7
--------------------------------------------------
CVE ID: CVE-2024-54096
Description: Vulnerability of improper access control in the MTP module
Impact: Successful exploitation of this vulnerability may affect integrity and accuracy.
Published Date: 2024-12-12T12:15:22.863
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 5.3
--------------------------------------------------
CVE ID: CVE-2024-54097
Description: Security vulnerability in the HiView module
Impact: Successful exploitation of this vulnerability may affect feature implementation and integrity.
Published Date: 2024-12-12T12:15:23.060
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 7.3
--------------------------------------------------
CVE ID: CVE-2024-54098
Description: Service logic error vulnerability in the system service module
Impact: Successful exploitation of this vulnerability may affect service integrity.
Published Date: 2024-12-12T12:15:23.243
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 8.5
--------------------------------------------------
CVE ID: CVE-2024-54099
Description: File replacement vulnerability on some devices
Impact: Successful exploitation of this vulnerability will affect integrity and confidentiality.
Published Date: 2024-12-12T12:15:23.420
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 6.7
--------------------------------------------------
CVE ID: CVE-2024-54100
Description: Vulnerability of improper access control in the secure input module
Impact: Successful exploitation of this vulnerability may cause features to perform abnormally.
Published Date: 2024-12-12T12:15:23.593
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 6.2
--------------------------------------------------
CVE ID: CVE-2024-54101
Description: Denial of service (DoS) vulnerability in the installation module
Impact: Successful exploitation of this vulnerability will affect availability.
Published Date: 2024-12-12T12:15:23.763
Vulnerability Status: Awaiting Analysis
reference uri: https://consumer.huawei.com/en/support/bulletin/2024/12/
Impact Score: 6.2
--------------------------------------------------
```

いろいろ思うところはあるけどとりあえず，動いたのでヨシ!

---

## デプロイ準備

### APIキーのハードコーディングをやめる

とりあえず，動けばヨシとはいえ，流石にAPIキーのハードコードはやめてLambdaにデプロイしたい。

1. .envを作成してapi_keyを退避
2. .gitignoreに.envを追加
3. serverlessでdotenvを使うためのプラグラインを入れる
    ```shell
    npm install serverless-dotenv-plugin
    ```
4. serverless.ymlにpluginの設定を追加
    ```serverless.yml
    plugins:
      - serverless-dotenv-plugin
    ```

### libraryの依存関係の解決

requestsは外部ライブラリなのでそのままlambdaにデプロイしてもimport errorになります。

1. requirements.txtを作成。
2. serverlessのプラグインをインストール
    ```shell
    npm install serverless-python-requirements
    ```
3. serverless.ymlにpluginの設定を追加
    ```serverless.yml
     plugins:
      - serverless-dotenv-plugin
      - serverless-python-requirements

    custom:
      pythonRequirements:
        dockerizePip: true
    ```

### timeoutの設定

一応30秒くらいにしておいた

```serverlsss.yml
provider:
  name: aws
  runtime: python3.12
  timeout: 30

  environment:
    NVD_API_KEY: ${env:NVD_API_KEY}

functions:
  rateHandler:
    handler: handler.run
    timeout: 30
    events:
     - schedule: rate(1 day)

```

---

## デプロイして検証

最終的なserverless.ymlも一応貼っておく

```serverless.yml
# "org" ensures this Service is used with the correct Serverless Framework Access Key.
org: sigma18
# "app" enables Serverless Framework Dashboard features and sharing them with other Services.
app: scrapecve
# "service" is the name of this project. This will also be added to your AWS resource names.
service: cvechecker

provider:
  name: aws
  runtime: python3.12
  timeout: 30

  environment:
    NVD_API_KEY: ${env:NVD_API_KEY}

functions:
  rateHandler:
    handler: handler.run
    timeout: 30
    events:
     - schedule: rate(1 day)
     # - schedule: rate(1 minute)

plugins:
  - serverless-dotenv-plugin
  - serverless-python-requirements

custom:
  pythonRequirements:
    dockerizePip: true


```

CloudWatchのログから見ると，printって一行ずつ出力されてしまうのね。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/aa5e32aa-c422-bb93-b621-c73b6ea60834.png)

次回に期待!

## 次回

通知設定と出力の整形をしたい

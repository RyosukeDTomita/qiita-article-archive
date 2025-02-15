# Qiita記事アーカイブ用リポジトリ

## アーカイブの初期設定

1. `qiita-cli`をインストールする

  ```shell
  npm install @qiita/qiita-cli --save-dev
  ```
2. https://qiita.com/settings/applications からtokenを取得する。
  > [!NOTE]
  > `qiita-cli`経由で編集する予定がないならread-onlyトークンで良い。
3. tokenでCLI経由でログインして記事を取得

  ```
  npx qiita login
  npx qiita pull
  ls public/
  ```

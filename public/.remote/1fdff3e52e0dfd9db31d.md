---
title: Copilot NES(NextEditSuggestions)のTab，ESCが効かない問題を解決する
tags:
  - Vim
  - GitHub
  - error
  - copilot
  - copilotnes
private: false
updated_at: '2025-05-06T22:11:31+09:00'
id: 1fdff3e52e0dfd9db31d
organization_url_name: null
slide: false
ignorePublish: false
---
## これは何?

[Copilot NES](https://learn.microsoft.com/ja-jp/shows/visual-studio-code/next-edit-suggestions-for-github-copilot-in-action)を使っていて

TabキーやEscキーがうまく反応しない事象に遭遇したので解決策を共有します。

↓以下の状況でTabやEscを押しても反応しない。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/7b26332a-4ec5-4ff2-8182-672043e60fa4.png)

---

## 解決方法

vscodevim.vimとキーバインドが競合していました。

### Tab

Tabに関しては，以下のキーバインドを消しました。自分はvimのinsertモード以外でTabキーを使うことがないので。

```json
{
  "key": "tab",
  "command": "extension.vim_tab",
  "when": "editorTextFocus && vim.active && !inDebugRepl && vim.mode != 'Insert'"
}
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/ef2b8ffb-b02a-488e-b3a1-5b19d1aa1721.png)
キーバインドの設定はコマンドパレットからキーマップの設定ファイルを開けます。

:::note warn
Open **Default** Keyboard Shortcuts(JSON)はreadonlyの別物なのでユーザの設定ファイルを変更する。
Ubuntuの場合，ユーザのキーバインドの設定ファイルは`~/.config/Code/User/keybindings.json`を直接開いて編集しても良い。
:::

これで一旦Tabを押してサジェストを受け入れられるようになりました。

### ESC

~~Escに関しては現状消すと不便すぎるので，NESのサジェストを無視してコードを書くことで対応しています。~~
~~(NESのサジェストを明示的にEscで拒否しなくても何かしらの文字を打てば消えるので)~~

~~もしくは，自分がCtrl [をEscの代わりに使う原理主義者になるか。~~

ありがたいことに自分の日本語の
[tweet](https://x.com/BrigitMurtaugh/status/1912559840305852663)を見て，開発者の人がコメントをくれました。

下記のIssueの通りに設定したところESCも動くようになりました。

https://github.com/VSCodeVim/Vim/issues/9459#issuecomment-2648156285

```json
  {
    "key": "escape",
    "command": "-extension.vim_escape",
    "when": "editorTextFocus && vim.active && !inDebugRepl"
  },
  {
    "key": "escape",
    "command": "extension.vim_escape",
    "when": "editorTextFocus && vim.active && !inDebugRepl && !testing.isPeekVisible && !testing.isInPeek && (vim.mode == 'Insert' || !notebookEditorFocused) && !inlineEditIsVisible && !suggestWidgetVisible && !findWidgetVisible && !dirtyDiffVisible"
  },
  {
    "key": "escape",
    "command": "runCommands",
    "when": "vim.mode == 'Insert' && inlineEditIsVisible",
    "args": {
      "commands": [
        "editor.action.inlineSuggest.hide",
        "extension.vim_escape"
      ]
    }
  },
  {
    "key": "escape",
    "command": "runCommands",
    "when": "vim.mode == 'Insert' && suggestWidgetVisible",
    "args": {
      "commands": [
        "hideSuggestWidget",
        "extension.vim_escape"
      ]
    }
  },
  {
    "key": "escape",
    "command": "-hideSuggestWidget",
    "when": "suggestWidgetVisible && textInputFocus"
  },
  {
    "key": "escape",
    "command": "hideSuggestWidget",
    "when": "suggestWidgetVisible && textInputFocus && !vim.active"
  },
```

---
title: vimmerに捧ぐ，tmux入門
tags:
  - tmux
private: false
updated_at: '2025-01-09T16:29:04+09:00'
id: 9ad98b9fed744bbd8cf7
organization_url_name: nri
slide: false
ignorePublish: false
---
## tmuxとは

> [とほほのtmux入門より](https://www.tohoho-web.com/ex/tmux.html)
>
> - ターミナルマルチプレクサ(Terminal Multiplexer) の略です。
> - Linux 系のターミナル画面を複数のセッション、ウィンドウ、ペインに分割して利用することができます。

つまり，一つのターミナルで複数の画面を開いて作業できるので，ssh先とかで作業する時に非常に便利です。

### それ複数ターミナルで良くない?を論破しつつ，tmuxのメリットを紹介

- ターミナルいっぱい開けば良くない? --> tmuxなら超細かく画面を分割できるうえに，alt tabで等で切り替えるアプリの数が増えない。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3718390/2e713603-62d2-72c6-4c3b-447e06195e79.png)


## tmuxとは

> [とほほのtmux入門より](https://www.tohoho-web.com/ex/tmux.html)
>
> - ターミナルマルチプレクサ(Terminal Multiplexer) の略です。
> - Linux 系のターミナル画面を複数のセッション、ウィンドウ、ペインに分割して利用することができます。

つまり，一つのターミナルで複数の画面を開いて作業できるので，ssh先とかで作業する時に非常に便利です。

### それ複数ターミナルで良くない?を論破しつつ，tmuxのメリットを紹介

- ターミナルいっぱい開けば良くない? --> tmuxなら超細かく画面を分割できるうえに，alt tabで等で切り替えるアプリの数が増えない。

![tmuxの例](/images/tmux.png)

- ショートカット覚えるのがめんどくさい。 --> ショートカットは自分好みにカスタマイズできる。
- え，それVSCodeで良くない? --> VSCodeのターミナルこそtmuxの真骨頂。小さなターミナルの空間を有効活用できる。

---

## 自分のやった設定紹介

いい使い方があればぜひ，コメント等で教えてほしいです。tmuxビギナーなので。

[自分の.tmux.conf](https://github.com/RyosukeDTomita/dotfiles/blob/main/.tmux.conf)

### 1. prefixキーの変更

tmuxではコマンドを実行する前にprefixキーを押下する必要があります。
デフォルトはctrl+bですが，私はctrl+qに変更してみました。

```
set -g prefix C-q
unbind C-b # C-bのキーバインドを解除
```

### 2. ウィンドウの移動をvim風に

- hjklでウィンドウを移動できるようにしました。
- HJKLでペインのリサイズができるようにしました(これはあんまり使わないかも)。

```
# vimのキーバインドでペインを移動する
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
# vimのキーバインドでペインをリサイズする
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
```

### 3. ペインの分割を-と\に変更

- 縦横の分割がわかりやすそうなうな-と\に変更しました。
- 本当は-と|にしたかったけど，|はshiftを押さないと入力できないので断念。

```
bind \\ split-window -h # \ でペインを縦分割する。NOTE: 本当は|が良かったが押しにくいので
bind - split-window -v # - でペインを縦分割する
```

### 4. マウス操作の有効化

部分的にマウスを使いたくなるときもありそうなので一応有効にしました。将来的には消していきたい。

```
set-option -g mouse on
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= '#{pane_in_mode}' 'send-keys -M' 'copy-mode -e'"
```

### 5. コピーをvim風に

viのようにターミナル上のコマンドやコマンドの結果をコピーできるので結構便利です。
自分はxselを使ってクリップボードにコピーしていますが，xclip等でも可能です。

```
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection # 'v' で選択を始める
bind -T copy-mode-vi V send -X select-line # 'V' で行選択
bind -T copy-mode-vi C-v send -X rectangle-toggle # 'C-v' で矩形選択
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "xsel -ib" # 'y' でヤンクしてクリップボードに
bind -T copy-mode-vi Y send -X copy-line-pipe-and-cancel "xsel -ib" # 'Y' で行ヤンクしてクリップボードに
bind-key C-p paste-buffer # 'C-p'でペースト
```

### 補足: vimのescの反応を早くする

デフォルトだとvimでescを押下してから1秒ほどラグがあるので反応早くします。

```
set -sg escape-time 1

```


- ショートカット覚えるのがめんどくさい。 --> ショートカットは自分好みにカスタマイズできる。
- え，それVSCodeで良くない? --> VSCodeのターミナルこそtmuxの真骨頂。小さなターミナルの空間を有効活用できる。

---

## 自分のやった設定紹介

いい使い方があればぜひ，コメント等で教えてほしいです。tmuxビギナーなので。

[自分の.tmux.conf](https://github.com/RyosukeDTomita/dotfiles/blob/main/.tmux.conf)

### 1. prefixキーの変更

tmuxではコマンドを実行する前にprefixキーを押下する必要があります。
デフォルトはctrl+bですが，私はctrl+qに変更してみました。

```
set -g prefix C-q
unbind C-b # C-bのキーバインドを解除
```

### 2. ウィンドウの移動をvim風に

- hjklでウィンドウを移動できるようにしました。
- HJKLでペインのリサイズができるようにしました(これはあんまり使わないかも)。

```
# vimのキーバインドでペインを移動する
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
# vimのキーバインドでペインをリサイズする
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
```

### 3. ペインの分割を-と\に変更

- 縦横の分割がわかりやすそうなうな-と\に変更しました。
- 本当は-と|にしたかったけど，|はshiftを押さないと入力できないので断念。

```
bind \\ split-window -h # \ でペインを縦分割する。NOTE: 本当は|が良かったが押しにくいので
bind - split-window -v # - でペインを縦分割する
```

### 4. マウス操作の有効化

部分的にマウスを使いたくなるときもありそうなので一応有効にしました。将来的には消していきたい。

```
set-option -g mouse on
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= '#{pane_in_mode}' 'send-keys -M' 'copy-mode -e'"
```

### 5. コピーをvim風に

viのようにターミナル上のコマンドやコマンドの結果をコピーできるので結構便利です。
自分はxselを使ってクリップボードにコピーしていますが，xclip等でも可能です。

```
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection # 'v' で選択を始める
bind -T copy-mode-vi V send -X select-line # 'V' で行選択
bind -T copy-mode-vi C-v send -X rectangle-toggle # 'C-v' で矩形選択
bind -T copy-mode-vi y send -X copy-pipe-and-cancel "xsel -ib" # 'y' でヤンクしてクリップボードに
bind -T copy-mode-vi Y send -X copy-line-pipe-and-cancel "xsel -ib" # 'Y' で行ヤンクしてクリップボードに
bind-key C-p paste-buffer # 'C-p'でペースト
```

### 補足: vimのescの反応を早くする

デフォルトだとvimでescを押下してから1秒ほどラグがあるので反応早くします。

```
set -sg escape-time 1
```

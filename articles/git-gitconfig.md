---
title: "【環境構築】gitの.gitconfigを設定してコマンドを使いやすくしよう！"
emoji: "🎉"
type: "tech"
topics: [git]
published: false
---
# はじめに
この記事は2022年のアドベントカレンダー用の記事である。
https://qiita.com/advent-calendar/2022/frick

今回のアドベントカレンダーでは以下の技術の話をして行きます。
Git × GitHub Actions × Docker × NginX ×
Elixir/Phoenix × PostgreSQL × TailWindCSS × DaisyUI × AlpineJS × 
NeosVR

もし何かの為になれば幸いです。

# .gitconfig
```
[user]
  name = FRICK
  email = xxx@xxx.com
[alias]
  ci = commit
  cia = commit --amend
  cian = commit --amend --no-edit
  co = checkout
  cop = checkout @{-1}
  cp = cherry-pick
  mf = merge --ff-only
  mfp = merge --ff-only @{-1}
  s = status
  st = status -s
  tip = log -1
  lo = log --oneline
  l  = log --oneline -5 --graph --decorate
  ll = log --oneline -20 --graph --decorate
  lll = log --oneline -40 --graph --decorate
  d  = diff
  dc = diff --cached
  dn = diff --name-status
  dh = diff HEAD^!
  dt = diff-tree -p
  fe = fetch -p
```
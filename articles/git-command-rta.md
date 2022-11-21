---
title: "【Git】作業RTAでよく使うコマンド"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git]
published: false
---
# はじめに
この記事は2022年のアドベントカレンダー用の記事である。
https://qiita.com/advent-calendar/2022/frick

今回のアドベントカレンダーでは以下の技術の話をして行きます。
Git × GitHub Actions × Docker × nginx ×
Elixir/Phoenix × PostgreSQL × TailWindCSS × DaisyUI × AlpineJS × 
NeosVR

もし何かの為になれば幸いです。

# Gitでよく使うコマンド

```
#GitHubで作ったrepositoryをPCにクローンする
git clone 

git status

git checkout master

git fetch

git pull

git merge --ff-only origin/master

if(! "Your branch is up to date with 'origin/master'." ){ // 掃除
    git checkout .
    git clean -df
}

git checkout -b issue-xxxx-hoge

git push -u origin issue-xxxx-hoge

git add .

git commit -m ""

git commit --amend

git push 

git push -f

git rebase master

git reset --hard @~1

git stash
```
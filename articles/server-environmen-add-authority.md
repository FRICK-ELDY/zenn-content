---
title: "【環境構築】ubuntuユーザ追加とsudo権限付与しよう！"
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

# ubuntu
### root以外のユーザ作成
``` :Terminal
adduser hoge
```
### SSH接続でのrootログイン禁止
ユーザーをsudoグループに追加
``` :Terminal
gpasswd -a hoge sudo
```
PAMの設定を追記。
sudo所属のユーザのみsuコマンドが実行できるように設定。
``` :/etc/pam.d/su
auth required pam_wheel.so group=adm
```
```  :/etc/login.defs
SU_WHEEL_ONLY yes
``` 
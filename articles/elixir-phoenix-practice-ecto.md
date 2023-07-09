---
title: "【Elixir/Pnoenix】練習:Ecto"
emoji: "🌟"
type: "tech"
topics: [elixir,phoenix,db,ecto]
published: false
---
# 環境

```:elixir -v
Erlang/OTP 25 [erts-13.0.4] [source] [64-bit] [smp:16:16] [ds:16:16:10] [async-threads:1] [jit:ns]

Elixir 1.13.4 (compiled with Erlang/OTP 25)
```
参照:Elixirのインストール
```:mix phx.new -v
Phoenix installer v1.6.12
```
参照:Phoenixのインストール
``` :psql --version
psql (PostgreSQL) 12.9 (Ubuntu 12.9-0ubuntu0.20.04.1)
```
参照:PostgreSQLの使い方
https://www.javadrive.jp/postgresql

# 手順
Phoenixプロジェクト作成
```:command
mix phx.new self_access_01
```

<details><summary>すごく長い文章とかプログラムとか</summary><div>

```
* creating self_access_01/config/config.exs
...
...
...
* creating self_access_01/priv/static/favicon.ico

Fetch and install dependencies? [Yn] y
* running mix deps.get
* running mix deps.compile

We are almost there! The following steps are missing:

    $ cd self_access_01

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```
</div></details>

ディレクトリに移動
```:command
cd self_access_01
```

Gitで管理する
```:command
git init
git status
--------------------------------------------
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .formatter.exs
        .gitignore
        README.md
        assets/
        config/
        lib/
        mix.exs
        mix.lock
        priv/
        test/

nothing added to commit but untracked files present (use "git add" to track)
--------------------------------------------

```

gitHubにアクセスし、新規リポジトリを作成する
https://github.com/new
https://github.com/FRICK-ELDY/self_access_01
```
git@github.com:FRICK-ELDY/self_access_01.git
```


# 余談
PostgreSQLをインストールするとOSにPostgreのユーザーが登録される。
```:command
su postgres // ユーザー切り替え
psql // SQLコマンド実行

psql (12.9 (Ubuntu 12.9-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# \q //プロンプト終了
```
PostgreSQLに登録されているユーザーを確認。

```:postgres=#
postgres=# select * from pg_user;
 usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig 
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 postgres |       10 | t           | t        | t       | t            | ******** |          | 
(1 row)

```
postgresユーザーが居なければ作成。

```
create role postgres with superuser login password '';
```
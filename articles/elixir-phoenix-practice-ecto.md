---
title: "【Elixir/Pnoenix】自主練:phx.gen.live"
emoji: "🌟"
type: "tech"
topics: [elixir,phoenix,db,ecto]
published: true
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
**Phoenixプロジェクト作成**
```
mix phx.new self_access_01
```
:::details phx.newの処理
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
:::
**Phoenixプロジェクトのディレクトリに移動**
```
cd self_access_01
```
**Git作成**
```
git init
```
:::details 現在のgit status
```
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
```
:::
**Gitに変更をaddしてcommit**
```
git add .
git commit -m "run mix phx.new"
```
**gitHubで新規リポジトリを作成**
https://github.com/new
```
git@github.com:FRICK-ELDY/self_access_01.git
```
https://github.com/FRICK-ELDY/self_access_01
**remoteに追加してGitHubにPush**
```
git remote add origin git@github.com:FRICK-ELDY/self_access_01.git
git push -u origin master
```
:::details 現在のgit status (この状態がスタート)
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```
:::
**データベースの作成**
```
mix ecto.create
```
:::details ecto.createの処理
```
warning: the :gettext compiler is no longer required in your mix.exs.

Please find the following line in your mix.exs and remove the :gettext entry:

    compilers: [..., :gettext, ...] ++ Mix.compilers(),

  (gettext 0.22.3) lib/mix/tasks/compile.gettext.ex:5: Mix.Tasks.Compile.Gettext.run/1
  (mix 1.13.4) lib/mix/task.ex:397: anonymous fn/3 in Mix.Task.run_task/3
  (mix 1.13.4) lib/mix/tasks/compile.all.ex:92: Mix.Tasks.Compile.All.run_compiler/2
  (mix 1.13.4) lib/mix/tasks/compile.all.ex:72: Mix.Tasks.Compile.All.compile/4
  (mix 1.13.4) lib/mix/tasks/compile.all.ex:59: Mix.Tasks.Compile.All.with_logger_app/2
  (mix 1.13.4) lib/mix/tasks/compile.all.ex:36: Mix.Tasks.Compile.All.run/1

Compiling 14 files (.ex)
Generated self_access_01 app
The database for SelfAccess01.Repo has been created
```
:::
**試にPhoenixサーバーを立ち上げてみる**
```
mix phx.server
```
localhost:4000にて
![](/images/phoenix/phx.new.png)
無事立ち上がりました！

**mix phx.gen.live を試してみる**
https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Live.html
サンプルより
```
mix phx.gen.live Accounts User users name:string age:integer
```
:::details phx.gen.liveの処理
```
* creating lib/self_access_01_web/live/user_live/show.ex
* creating lib/self_access_01_web/live/user_live/index.ex
* creating lib/self_access_01_web/live/user_live/form_component.ex
* creating lib/self_access_01_web/live/user_live/form_component.html.heex
* creating lib/self_access_01_web/live/user_live/index.html.heex
* creating lib/self_access_01_web/live/user_live/show.html.heex
* creating test/self_access_01_web/live/user_live_test.exs
* creating lib/self_access_01_web/live/live_helpers.ex
* creating lib/self_access_01/accounts/user.ex
* creating priv/repo/migrations/20230709025908_create_users.exs
* creating lib/self_access_01/accounts.ex
* injecting lib/self_access_01/accounts.ex
* creating test/self_access_01/accounts_test.exs
* injecting test/self_access_01/accounts_test.exs
* creating test/support/fixtures/accounts_fixtures.ex
* injecting test/support/fixtures/accounts_fixtures.ex
* injecting lib/self_access_01_web.ex

Add the live routes to your browser scope in lib/self_access_01_web/router.ex:

    live "/users", UserLive.Index, :index
    live "/users/new", UserLive.Index, :new
    live "/users/:id/edit", UserLive.Index, :edit

    live "/users/:id", UserLive.Show, :show
    live "/users/:id/show/edit", UserLive.Show, :edit


Remember to update your repository by running migrations:

    $ mix ecto.migrate
```
:::

```Elixir:lib/self_access_01_web/router.ex 
defmodule SelfAccess01Web.Router do
  use SelfAccess01Web, :router
  ...(略)...
  scope "/", SelfAccess01Web do
    pipe_through :browser
    # ココから追加
    live "/users", UserLive.Index, :index
    live "/users/new", UserLive.Index, :new
    live "/users/:id/edit", UserLive.Index, :edit

    live "/users/:id", UserLive.Show, :show
    live "/users/:id/show/edit", UserLive.Show, :edit
    # ココまで
    get "/", PageController, :index
  end
  ...(略)...
end
```
**データベースのmigrate**
```
mix ecto.migrate
```
localhost:4000/users を確認
![](/images/phoenix/phx.gen.live.png)
追加されてた！
![](/images/phoenix/phx.gen.live2.png) 
NewUserも出来る！

**GitHubにPush**
折角なので、Issueの作成からプルリク、コミットまで。
https://github.com/FRICK-ELDY/self_access_01/issues/1
ブランチを切ってリモートにPushする ※本当はブランチを切ってから実装する！大事！
```
git checkout -b frick/issue-1-run-phx_gen_live
git push -u origin frick/issue-1-run-phx_gen_live
```

:::details 現在のgit status ブランチが切り変わっている事を確認
```
On branch frick/issue-1-run-phx_gen_live
Your branch is up to date with 'origin/frick/issue-1-run-phx_gen_live'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   lib/self_access_01_web.ex
        modified:   lib/self_access_01_web/router.ex

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        lib/self_access_01/accounts.ex
        lib/self_access_01/accounts/
        lib/self_access_01_web/live/
        priv/repo/migrations/20230709025908_create_users.exs
        test/self_access_01/
        test/self_access_01_web/live/
        test/support/fixtures/

no changes added to commit (use "git add" and/or "git commit -a")
```
:::
```
git add .
git commit -m "run mix.gen.live and fix router.ex"
git push
```
プルリクの作成 (コメントにClone #issue-no を付けると、マージ者が楽)
https://github.com/FRICK-ELDY/self_access_01/pull/2
コード修正内容を確認 (このコードが自動で作成されるの凄い！！！)
https://github.com/FRICK-ELDY/self_access_01/pull/2/files
※本来であれば、ReviewでApproveするのだが、個人開発なので割愛。
問題ないのでmerge＆Delete Branch
開発環境を最新に更新
```
git checkout master
git fetch
git pull
(git merge --ff-only origin/master)
```
:::details git status masterが最新であることを確認
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```
:::

**iexでデータベースのリストを使って遊んでみる**
iexを起動
```
iex
-----------
Erlang/OTP 25 [erts-13.0.4] [source] [64-bit] [smp:16:16] [ds:16:16:10] [async-threads:1] [jit:ns]

Interactive Elixir (1.13.4) - press Ctrl+C to exit (type h() ENTER for help)
```
`lib\self_access_01\accounts.ex` を参考にする

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
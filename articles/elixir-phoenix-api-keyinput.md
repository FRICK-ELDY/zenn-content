---
title: "【Elixir/Pnoenix】key_input api"
emoji: "🌟"
type: "tech"
topics: [elixir,phoenix,db,ecto]
published: true
---

# 環境
Windows 11 + WSL2 環境
```:elixir -v
Erlang/OTP 27 [erts-15.0] [source] [64-bit] [smp:32:32] [ds:32:32:10] [async-threads:1] [jit:ns]

Elixir 1.16.3 (compiled with Erlang/OTP 26)
```
```:mix phx.new -v
Phoenix installer v1.7.12
```
# 手順
## ①プロジェクト作成
作業ディレクトリに作成 or 移動
```:command
mkdir {作業ディレクトリ}
cd {作業ディレクトリ}
```
Phoenix Projectを作成し、ディレクトリに移動
```:command
mix phx.new key_input_api --no-ecto
cd key_input_api
```
Phoenixが立ち上がることを確認
```:command
mix phx.server
```
`http://127.0.0.1:4000/` にアクセスし、正常に立ち上がることを確認
## ②キー入力を取得するLiveViewを作成
`lib/key_input_api_web/`に`live`フォルダを作成
`lib/key_input_api_web/live/key_input_live.ex`を新規作成
```elixir
defmodule KeyInputApiWeb.KeyInputLive do
  use KeyInputApiWeb, :live_view

  def mount(_params, _session, socket) do
    {:ok, assign(socket, :key_input, "")}
  end

  def handle_event("key_press", %{"key" => key}, socket) do
    {:noreply, assign(socket, :key_input, key)}
  end

  def handle_event("key_up", %{"key" => key}, socket) do
    {:noreply, assign(socket, :key_input, key)}
  end
end
```
`lib/key_input_api_web/live/key_input_live.html.leex`を新規作成
```html
<div id="key-input-container" phx-hook="KeyInputHook" tabindex="0">
  <h1>Press any key</h1>
  <p>Key pressed: <%= @key_input %></p>
</div>
```
`assets/js/app.js`を編集: Hooksの設定
```javascript
(...省略...)
import {Socket} from "phoenix"
import {LiveSocket} from "phoenix_live_view"
import topbar from "../vendor/topbar"

let Hooks = {}
Hooks.KeyInputHook = {
  mounted() {
    this.el.addEventListener("keydown", (e) => {
      this.pushEvent("key_press", { key: e.key })
    })
    this.el.addEventListener("keyup", (e) => {
      this.pushEvent("key_up", { key: "" })
    })
  }
}

let csrfToken = document.querySelector("meta[name='csrf-token']").getAttribute("content")
let liveSocket = new LiveSocket("/live", Socket, {
  longPollFallbackMs: 2500,
  params: {_csrf_token: csrfToken},
  hooks: Hooks
})
(...省略...)
```
`lib/key_input_api_web/router.ex`を編集: エンドポイントを追加
```Elixir
defmodule KeyInputApiWeb.Router do
  (...省略...)
  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", KeyInputApiWeb do
    pipe_through :browser

    get "/", PageController, :home
+   live "/key_input", KeyInputLive, :index
  end
  (...省略...)
end
```
`http://127.0.0.1:4000/key_input`にアクセスして、key_inputが表示されたらOK
## ③Genserverでキー入力を保存
`lib/key_input_api_web/key_input_storage.ex`を作成
```Elixir
defmodule KeyInputApiWeb.KeyInputStorage do
  use GenServer

  def start_link(_) do
    GenServer.start_link(__MODULE__, "", name: __MODULE__)
  end

  def init(initial_key_input) do
    {:ok, initial_key_input}
  end

  def set_key_input(key_input) do
    GenServer.call(__MODULE__, {:set_key_input, key_input})
  end

  def get_key_input do
    GenServer.call(__MODULE__, :get_key_input)
  end

  def handle_call({:set_key_input, key_input}, _from, _state) do
    {:reply, :ok, key_input}
  end

  def handle_call(:get_key_input, _from, state) do
    {:reply, state, state}
  end
end
```
`lib/key_input_api/application.ex`の編集: 上記関数をアプリケーションに追加
```Elixir
def start(_type, _args) do
  children = [
    KeyInputApiWeb.Telemetry,
    {DNSCluster, query: Application.get_env(:key_input_api, :dns_cluster_query) || :ignore},
    {Phoenix.PubSub, name: KeyInputApi.PubSub},
    # Start the Finch HTTP client for sending emails
    {Finch, name: KeyInputApi.Finch},
    # Start a worker by calling: KeyInputApi.Worker.start_link(arg)
    # {KeyInputApi.Worker, arg},
    # Start to serve requests, typically the last entry
    KeyInputApiWeb.Endpoint,
    KeyInputApiWeb.KeyInputStorage # ココ
  ]

  # See https://hexdocs.pm/elixir/Supervisor.html
  # for other strategies and supported options
  opts = [strategy: :one_for_one, name: KeyInputApi.Supervisor]
  Supervisor.start_link(children, opts)
end
```
`lib/key_input_api_web/live/key_input_live.ex`を編集: `set_key_input`追加
```Elixir
defmodule KeyInputApiWeb.KeyInputLive do
  use KeyInputApiWeb, :live_view

  def mount(_params, _session, socket) do
    {:ok, assign(socket, :key_input, "")}
  end

  def handle_event("key_press", %{"key" => key}, socket) do
    KeyInputApiWeb.KeyInputStorage.set_key_input(key) # ココ
    {:noreply, assign(socket, :key_input, key)}
  end

  def handle_event("key_up", %{"key" => key}, socket) do
    KeyInputApiWeb.KeyInputStorage.set_key_input(key) # ココ
    {:noreply, assign(socket, :key_input, key)}
  end
end
```
アプリケーションを変更したので、assets.deployをする。
```:command
mix assets.deploy
```
これで、入力したキーがGenserverに保存される。
## ④API取得用のエンドポイントを作成
`key_input_api/lib/key_input_api_web/controllers/key_input_controller.ex`
を新規作成し、コードを書く。
```Elixir
defmodule KeyInputApiWeb.KeyInputController do
  use KeyInputApiWeb, :controller

  def get_key_input(conn, _params) do
    key_input = KeyInputApiWeb.KeyInputStorage.get_key_input()
    json(conn, %{key_input: key_input})
  end
end
```
`lib/key_input_api_web/router.ex`を編集: APIのエンドポイントを追加
```Elixir
defmodule KeyInputApiWeb.Router do
  (...省略...)

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", KeyInputApiWeb do
    pipe_through :browser

    get "/", PageController, :home
    live "/key_input", KeyInputLive, :index
  end

  scope "/api/v1", KeyInputApiWeb do
    pipe_through :api
    get "/getkeyinput", KeyInputController, :get_key_input
  end

  (...省略...)
end

```
`http://127.0.0.1:4000/api/v1/getkeyinput` にアクセスし、
json形式で`{"key_input":""}`が表示されたらOK

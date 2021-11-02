---
title: OTP:サーバ メモ
date: "2021-11-02T22:52:03.284Z"
description: "Elixir OTP:Server Memo"
---

『プログラミングElixir』(第1版)の「第16章 OTP:サーバ」を読んだメモを記していく。

# GenServerコールバック

GenServerには6つのコールバックがある。  
`use Genserver`を使うことによって、ユーザーは実装する必要のあるコーバルックだけ実装すれば良い。

## init(start_arguments)

新しいサーバ開始時にGenServerから呼ばれる。  
サーバ開始に成功すれば`{:ok, state}`を返し、開始できないなら`{:stop, reason}`を返す。  
タイムアウトの指定も可能。

## handle_call(request, from, state)

クライアントが`GenServer.call(pid, request)`を使った時に呼ばれる。  
requestパラメータは、リクエストの種類をアトムで渡す(`:next_number`, `:set_number`など)  
fromパラメータは、クライアントのPIDとユニークタグを含むタプル。  
stateパラメータは、サーバの状態を持つ。

成功時には`{:reply, result, new_state}`を返す。  
実装されていないrequest種類が呼ばれると`:bad_call`エラーを返す。


## handle_cast(request, state)

`GenServer.cast(pid, request)`を使ったときに呼ばれる。  
レスポンスが必要ない処理を実装する際に用いる。  
成功時は`{:noreply, new_state}`や`{:stop, reason, new_state}`を返す。


## handle_info(info, state)

timeoutメッセージ、リンクしたプロセスからの終了メッセージ、sendを用いてPIDへ送られたメッセージなど、callやcastリクエスト以外でやってくるメッセージを処理する。  


## terminate(reason, state)

サーバが終了しようとするときに呼ばれる。
サーバに監視をつけてしまえば、これについて考える必要はない(要調査)

## code_change(from_version, state, extra)

OTPが、走っているサーバを古いバージョンから新しいバージョンへ(無停止で)置き換える際に呼ばれる。  
古い状態のフォーマットを新しいフォーマットへ変換するなどの処理を行う

## format_status(reason, [pdict, state])

`:sys.get_status pid`などサーバの状態の表示する際に表示方法をカスタマイズするために使う。


## callとcastのレスポンス

callとcastのレスポンスには、いくつかのオプションを返すことができる

- **:hibernate**  
`:hibernate`をレスポンスに含めた場合、次のリクエストまでメモリ上からサーバの状態を取り除き、次のリクエストがあれば状態を読み込んで復活する。  
メモリを節約できるがCPUを余分に使う。


- **タイムアウトオプション**  
`:infinite`かミリ秒を渡してタイムアウト時間を指定できる。  
ミリ秒が渡された場合は、その時間までにサーバが何もしない(次のリクエストを送らない？)ならタイムアウトメッセージが送られる。

### レスポンスのパターン

下記二つはcall, cast共通

```
{:noreply, new_state [, :hibernate | timeout]}

{:stop, reason, new_state} // サーバに停止の合図をする
```

下記二つはhandle_callだけがレスポンスできる。

```
{:reply, response, new_state [ , :hibernate | timeout]} // クライアントにresponseを送る

{:stop, reason, reply, new_state} // レスポンスを送り、サーバに停止の合図をする。
```


# プロセスの名前を付ける


---
title: "PostgreSQLでプロセス確認とプロセスkill"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PostgreSQL", "SQL"]
published: true
---

最近、PostgreSQL のロックに遭遇することが頻発してる。
備忘もかねて、プロセスの確認と kill する方法を残しておく。

# 対象

- PostgreSQL でプロセスを確認したい人
- PostgreSQL で処理を kill したい人

# 環境

- PostgreSQL12

# プロセス確認

SQL の解説も書いているので興味があれば参考に。

```js:sql
select
  sel.pid as プロセス ID,
  sel.start as 開始時刻,
  sel.sql as 実行 SQL
from
  (
    select
      pg_stat_get_backend_pid(sgbi.bid) as pid,
      pg_stat_get_backend_activity_start(sgbi.bid) as start,
      pg_stat_get_backend_activity(sgbi.bid) as sql
    from
      (
        select
        pg_stat_get_backend_idset() as bid
      ) as sgbi
  ) as sel
where
  sel.sql <> '' -- 現在実行中の SQL が存在する場合のみ取得
  -- and
  -- sel.procpid ='' --　プロセス ID が特定できればここで指定
order by
  pid desc; -- プロセス ID 順にソート
;
```

**SQL の解説**

各情報は [統計情報関数](<[リンクのURL](https://www.postgresql.jp/document/12/html/monitoring-stats.html)>)を使って取得する。
ポイントとなる部分をピックアップして解説。

| 行  | コード                             | 説明                                             |
| --- | ---------------------------------- | ------------------------------------------------ |
| 8   | pg_stat_get_backend_pid            | バックエンド ID 番号に紐付くプロセス ID を取得   |
| 9   | pg_stat_get_backend_activity_start | バックエンド ID 番号に紐付く処理の開始時刻を取得 | Text |
| 10  | pg_stat_get_backend_activity       | バックエンド ID 番号を取得                       |
| 14  | select pg_stat_get_backend_idset   | バックエンド ID 番号に紐付く実行 SQL を取得      |

# プロセスを kill する方法

kill する方法は 2 パターン。
kill できる点では同じだが、挙動は異なるので注意。

## pg_cancel_backend で kill する

実行中の SQL をキャンセルする。
セッションが残っているので、データベースへ再接続が不要。

```js:sql
SELECT pg_cancel_backend(int:プロセス ID);
```

プロセス ID：プロセス確認で取得したプロセス ID

## pg_terminate_backend で kill する

セッションを切断する。
セッションが切れるので、データベースへ再接続が必要。

```js:sql
SELECT pg_terminate_backend(int:プロセス ID);
```

プロセス ID：プロセス確認で取得したプロセス ID

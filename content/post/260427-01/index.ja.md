---
title: PostgreSQLでよく使うコマンド集
date: 2026-04-27
description: PostgreSQLでよく使うコマンド集
tags: 
    - PostgreSQL
    - コマンド一覧
categories:
    - PostgreSQL
---
# PostgreSQLでよく使うコマンド集

PostgreSQL(`psql`)で実務でよく使うコマンドをまとめました。

## 1. 接続・終了

### 接続

```bash
# 基本形
psql -U ユーザー名 -d データベース名

# ホスト・ポート指定
psql -h localhost -p 5432 -U appuser -d appdb

# パスワード入力プロンプトを表示
psql -U appuser -d appdb -W

# Docker Compose経由
docker compose exec postgres psql -U appuser -d appdb
```

### 終了

```sql
\q              -- psqlを終了
```

## 2. 情報を一覧表示する `\d` 系コマンド

最も使用頻度が高いカテゴリです。

| コマンド | 内容 |
|---------|------|
| `\l` | データベース一覧 |
| `\dn` | スキーマ一覧 |
| `\dt` | テーブル一覧 |
| `\dt+` | テーブル一覧(サイズ付き) |
| `\dv` | ビュー一覧 |
| `\dm` | マテリアライズドビュー一覧 |
| `\di` | インデックス一覧 |
| `\ds` | シーケンス一覧 |
| `\df` | 関数一覧 |
| `\du` | ユーザー(ロール)一覧 |
| `\dp` | テーブル権限一覧 |
| `\d` | 全リレーション一覧 |
| `\d テーブル名` | テーブルの構造を表示 |
| `\d+ テーブル名` | テーブル構造の詳細(サイズ・統計含む) |

### 使用例

```sql
-- データベース一覧
\l

-- 現在のDBのテーブル一覧
\dt

-- usersテーブルの構造
\d users

-- 詳細情報付き
\d+ users
```

## 3. データベース・スキーマの切り替え

```sql
\c データベース名              -- データベースを切り替え
\c mydb appuser               -- DBとユーザーを同時に変更
\c                            -- 現在の接続情報を表示

SET search_path TO myschema;  -- スキーマを切り替え
SHOW search_path;             -- 現在のスキーマパス確認
```

## 4. クエリの基本(SQL)

### SELECT

```sql
-- 全件取得
SELECT * FROM users;

-- 条件付き
SELECT id, name FROM users WHERE age >= 20;

-- 並び替え・件数制限
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 件数を数える
SELECT COUNT(*) FROM users;

-- グループ化
SELECT status, COUNT(*) FROM orders GROUP BY status;
```

### INSERT / UPDATE / DELETE

```sql
-- 挿入
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- 複数行挿入
INSERT INTO users (name, email) VALUES
  ('Bob', 'bob@example.com'),
  ('Carol', 'carol@example.com');

-- 更新
UPDATE users SET email = 'new@example.com' WHERE id = 1;

-- 削除
DELETE FROM users WHERE id = 1;

-- 全削除(高速)
TRUNCATE TABLE users;
```

## 5. テーブル操作(DDL)

```sql
-- テーブル作成
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- テーブル削除
DROP TABLE users;
DROP TABLE IF EXISTS users;

-- カラム追加
ALTER TABLE users ADD COLUMN age INT;

-- カラム削除
ALTER TABLE users DROP COLUMN age;

-- カラム名変更
ALTER TABLE users RENAME COLUMN name TO username;

-- 型変更
ALTER TABLE users ALTER COLUMN age TYPE BIGINT;

-- テーブル名変更
ALTER TABLE users RENAME TO members;
```

## 6. トランザクション

```sql
BEGIN;                  -- トランザクション開始
UPDATE users SET ...;
DELETE FROM ...;
COMMIT;                 -- 確定

-- 取り消したいとき
BEGIN;
UPDATE users SET ...;
ROLLBACK;               -- 取り消し
```

## 7. インポート・エクスポート

### `\copy`(クライアント側のファイル)

```sql
-- CSVから取り込み
\copy users FROM '/path/to/users.csv' WITH CSV HEADER;

-- CSVへ出力
\copy users TO '/path/to/output.csv' WITH CSV HEADER;

-- クエリ結果をCSVへ
\copy (SELECT * FROM users WHERE active = true) TO 'active.csv' WITH CSV HEADER;
```

### コマンドラインから(`pg_dump` / `psql`)

```bash
# データベース全体をダンプ
pg_dump -U appuser appdb > backup.sql

# 特定のテーブルだけ
pg_dump -U appuser -t users appdb > users.sql

# リストア
psql -U appuser appdb < backup.sql
```

## 8. クエリ実行と結果操作

### SQLファイルを実行

```sql
\i /path/to/script.sql        -- ファイルを実行
\ir relative/path.sql         -- 現在のスクリプトからの相対パス
```

### 結果をファイルへ出力

```sql
\o /path/to/output.txt        -- 出力先をファイルに変更
SELECT * FROM users;
\o                            -- 出力先を画面に戻す
```

### 結果の表示形式

```sql
\x                            -- 拡張表示モード(縦長)切り替え
\x on                         -- 強制ON
\x off                        -- 強制OFF
\pset format aligned          -- 整形表示(デフォルト)
\pset format csv              -- CSV形式
\pset null '(NULL)'           -- NULLの表示文字を変更
```

`\x` の効果(横長のテーブルを縦に表示):

```
-[ RECORD 1 ]----+--------------------
id               | 1
name             | Alice
email            | alice@example.com
created_at       | 2025-01-15 10:30:00
```

## 9. クエリ実行計画(パフォーマンス確認)

```sql
-- 実行計画を表示(実行はしない)
EXPLAIN SELECT * FROM users WHERE email = 'a@example.com';

-- 実際に実行して計測
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@example.com';

-- より詳細に
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) SELECT ...;
```

## 10. インデックス操作

```sql
-- インデックス作成
CREATE INDEX idx_users_email ON users(email);

-- ユニークインデックス
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- 複合インデックス
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- インデックス削除
DROP INDEX idx_users_email;

-- インデックス一覧
\di
```

## 11. ユーザー・権限

```sql
-- ユーザー作成
CREATE USER newuser WITH PASSWORD 'password';

-- スーパーユーザー作成
CREATE USER admin WITH SUPERUSER PASSWORD 'password';

-- データベース作成
CREATE DATABASE mydb OWNER appuser;

-- 権限付与
GRANT ALL PRIVILEGES ON DATABASE mydb TO appuser;
GRANT SELECT ON users TO readonlyuser;
GRANT ALL ON ALL TABLES IN SCHEMA public TO appuser;

-- パスワード変更
ALTER USER appuser WITH PASSWORD 'newpassword';

-- 削除
DROP USER newuser;
DROP DATABASE mydb;
```

## 12. 便利なシステム情報コマンド

```sql
-- バージョン確認
SELECT version();

-- 現在の接続情報
\conninfo

-- 現在の日時
SELECT NOW();
SELECT CURRENT_DATE;

-- 現在のユーザー
SELECT CURRENT_USER;
SELECT SESSION_USER;

-- 現在のデータベース
SELECT current_database();

-- テーブルのサイズ
SELECT pg_size_pretty(pg_total_relation_size('users'));

-- データベースのサイズ
SELECT pg_size_pretty(pg_database_size('mydb'));

-- 実行中のクエリ一覧
SELECT pid, query, state FROM pg_stat_activity;

-- クエリのキャンセル(問題のあるクエリを止める)
SELECT pg_cancel_backend(pid);
SELECT pg_terminate_backend(pid);   -- より強力(セッション切断)
```

## 13. 履歴・ヘルプ

```sql
\?              -- メタコマンドのヘルプ
\h              -- SQL文のヘルプ一覧
\h SELECT       -- SELECT文のヘルプ
\h CREATE TABLE -- CREATE TABLEのヘルプ

-- 履歴
↑↓キー          -- 過去のコマンドを呼び出し
\s              -- コマンド履歴を表示
\s history.txt  -- 履歴をファイルに保存
```

## 14. 入力中のリセット

```sql
\r              -- 入力中のクエリをリセット(クリア)
\g              -- 直前のクエリを再実行
\gx             -- 直前のクエリを拡張表示で実行
```

## 15. 編集系

```sql
\e              -- 直前のクエリをエディタで編集
\e ファイル名    -- ファイルをエディタで編集して実行
\ef 関数名       -- 関数をエディタで編集
```

## 16. 実務でよく使うクエリ集

### テーブルの行数を一覧

```sql
SELECT
  schemaname,
  relname AS table_name,
  n_live_tup AS row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;
```

### テーブルサイズの一覧

```sql
SELECT
  tablename,
  pg_size_pretty(pg_total_relation_size('public.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size('public.'||tablename) DESC;
```

### カラム情報を取得

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'users';
```

### 実行中のクエリを確認

```sql
SELECT
  pid,
  age(clock_timestamp(), query_start) AS duration,
  usename,
  query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;
```

### ロック状態の確認

```sql
SELECT * FROM pg_locks WHERE NOT granted;
```

## 17. コマンドラインから直接実行

`psql`に入らずワンライナーで実行する方法。スクリプトや調査で便利。

```bash
# クエリを直接実行
psql -U appuser -d appdb -c "SELECT COUNT(*) FROM users;"

# メタコマンドを実行
psql -U appuser -d appdb -c "\dt"

# SQLファイルを実行
psql -U appuser -d appdb -f script.sql

# 結果をCSVで出力
psql -U appuser -d appdb -c "COPY users TO STDOUT WITH CSV HEADER" > users.csv
```

## 頻度別おすすめコマンド

### 毎日使うレベル(必須)

```sql
\dt             -- テーブル一覧
\d テーブル名    -- テーブル構造
\l              -- DB一覧
\c DB名         -- DB切り替え
\q              -- 終了
```

### 週に数回使うレベル

```sql
\di             -- インデックス一覧
\du             -- ユーザー一覧
\x              -- 拡張表示
\timing         -- 実行時間表示ON/OFF
\i ファイル名    -- SQLファイル実行
```

### 困ったとき・調査時

```sql
\?              -- メタコマンドヘルプ
\h SQL文        -- SQL構文ヘルプ
EXPLAIN ANALYZE -- 実行計画
\conninfo       -- 接続情報確認
```

## まとめ

PostgreSQLで**最低限覚えておくべきコマンド**は次の8つです。

```sql
\l              -- DB一覧
\c DB名         -- DB切り替え
\dt             -- テーブル一覧
\d テーブル名    -- テーブル構造
\du             -- ユーザー一覧
\x              -- 表示形式切り替え
\?              -- ヘルプ
\q              -- 終了
```

この8つに加え、SQL(`SELECT`/`INSERT`/`UPDATE`/`DELETE`)が使えれば、日常的な操作はほぼカバーできます。あとは必要に応じて `\copy`、`EXPLAIN ANALYZE`、`pg_dump` などを学んでいけば実務レベルで困りません。
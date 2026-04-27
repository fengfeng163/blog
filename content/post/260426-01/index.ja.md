---
title: UbuntuにPostgreSQLデータベースのインストールのベストプラクティス
date: 2026-04-27
description: UbuntuにPostgreSQLデータベースのインストールのベストプラクティス
tags: 
    - PostgreSQL
    - ベストプラクティス
    - インストール
    - Ubuntu
    - docker
categories:
    - 環境構築
---

# 規模の想定

以下は Ubuntu 上で Docker Compose を使って PostgreSQL DB を構築する際の実用的なベストプラクティスです。  
開発用途と**小〜中**規模本番用途を想定しています。

# 参照
参照1：ChatGpt 5.5 (メイン参照対象)
参照2：[【2025年度最新】PostgreSQL18をDocker使って日本語対応でインストール](https://bsblog.casareal.co.jp/archives/13408)

---

# 1. ディレクトリ構成

おすすめは `/opt` 配下です。

```bash
/opt/postgres/
├── compose.yaml
├── .env
├── .env.example
├── .gitignore
├── init/
│   └── 01-init.sql
├── backup/
└── README.md
```

PostgreSQL のデータ本体は、基本的には **Docker volume** に置くのがおすすめです。

```yaml
volumes:
  postgres_data:
```

ホスト側のディレクトリに直接置く bind mount も可能ですが、権限問題が起きやすいため、まずは named volume が無難です。

---

# 2. 推奨 `compose.yaml`

```yaml
services:
  postgres:
    image: postgres:18.3
    container_name: postgres
    restart: unless-stopped

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      TZ: Asia/Tokyo
      PGTZ: Asia/Tokyo
      # 認証方式を強化
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --locale=C --auth-host=scram-sha-256"
      POSTGRES_HOST_AUTH_METHOD: scram-sha-256
    volumes:
      # 名前付きボリュームでデータ永続化
      - postgres_data:/var/lib/postgresql/data
      # 初期化SQL(初回起動時のみ実行)
      - ./init:/docker-entrypoint-initdb.d:ro
      # バックアップ用ディレクトリ
      - ./backups:/backups
    ports:
      - "127.0.0.1:5432:5432"

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

    shm_size: 256mb

    networks:
      - db_net
    # リソース制限(本番向け)
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
    # ログ設定
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  postgres_data:
    driver: local

networks:
  db_net:
    driver: bridge
```

---

# 3. `.env` に秘密情報を分離する

`.env` を作成します。

```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=change_me_to_a_long_random_password
```

権限を絞ります。

```bash
chmod 600 .env
```

`.env` は Git に入れないでください。

```bash
echo ".env" >> .gitignore
echo "backups/" >> .gitignore
```

.env.example(Git管理する)
```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=
```

---

# 4. ポート公開は最小限にする

PostgreSQL を外部公開する必要がないなら、これがおすすめです。

```yaml
ports:
  - "127.0.0.1:5432:5432"
```

これは Ubuntu ホスト自身からのみ接続可能にする設定です。

もし Docker 内のアプリケーションからしか接続しないなら、`ports` 自体を消してもよいです。

```yaml
# ports は不要
```

同じ Compose ネットワーク上のアプリからは以下で接続できます。

```text
host: postgres
port: 5432
database: app_db
user: app_user
password: .envで指定した値
```

---

# 5. 初期化 SQL は `init/` に置く

初回起動時だけ実行したい SQL はここに置きます。

```bash
mkdir -p init
```

例：

```sql
-- init/01-init.sql
-- 拡張機能の有効化
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";

-- 読み取り専用ユーザー作成例
CREATE USER readonly WITH PASSWORD 'readonly_password';
GRANT CONNECT ON DATABASE appdb TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

CREATE TABLE IF NOT EXISTS users (
    id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
    name text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);
```

注意点として、`/docker-entrypoint-initdb.d` の中身は **データベースが空の初回起動時のみ**実行されます。

すでに volume が作成済みの場合、後から SQL を置いても自動実行されません。

やり直す場合：

```bash
docker compose down -v
docker compose up -d
```

ただし `-v` は DB データを削除します。

---

# 6. データ保存は named volume を使う

推奨：

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

確認：

```bash
docker volume ls
```

バックアップ時などに場所を確認するには：

```bash
docker volume inspect postgres_data
```

Compose のプロジェクト名によって、実際の volume 名は以下のようになることがあります。

```text
postgres_postgres_data
```

---

# 7. バックアップを必ず用意する

DB はコンテナよりも **バックアップ設計**が重要です。

## 手動バックアップ

```bash
mkdir -p backup
docker compose exec -T postgres pg_dump \
  -U "$POSTGRES_USER" \
  -d "$POSTGRES_DB" \
  > backup/app_db_$(date +%Y%m%d_%H%M%S).sql
```

ただし、上のコマンドはホスト側のシェルで `.env` が読み込まれていないと変数が空になります。

確実にやるなら：

```bash
set -a
source .env
set +a

docker compose exec -T postgres pg_dump \
  -U "$POSTGRES_USER" \
  -d "$POSTGRES_DB" \
  > backup/app_db_$(date +%Y%m%d_%H%M%S).sql
```

---

## リストア

```bash
set -a
source .env
set +a

cat backup/app_db_20260101_120000.sql | docker compose exec -T postgres psql \
  -U "$POSTGRES_USER" \
  -d "$POSTGRES_DB"
```

---

# 8. 本番では `latest` を使わない

避ける：

```yaml
image: postgres:latest
```

推奨：

```yaml
image: postgres:18
```

さらに厳密にするならマイナーバージョンまで固定します。

```yaml
image: postgres:18.3
```

理由は、意図せず PostgreSQL のメジャーバージョンが上がるとデータディレクトリの互換性問題が起きる可能性があるためです。

---

# 9. `container_name` は好みでよいが、本番では省略もあり

単体運用なら分かりやすいです。

```yaml
container_name: postgres
```

ただし、複数環境やスケール、同一ホストに複数 Compose プロジェクトを置く場合は名前衝突の原因になります。

その場合は省略して、Compose に任せる方が安全です。

---

# 10. アプリ用ユーザーと管理用ユーザーを分ける

`POSTGRES_USER` は初期スーパーユーザーに近い権限を持ちます。  
本番寄りの構成なら、アプリ専用ユーザーを作るのがおすすめです。

例：

```sql
-- init/02-create-app-user.sql

CREATE USER app_user_limited WITH PASSWORD 'change_this_password';

CREATE DATABASE app_db OWNER app_user_limited;

GRANT CONNECT ON DATABASE app_db TO app_user_limited;
```

ただし、`POSTGRES_DB=app_db` と重複して DB を作るとエラーになる可能性があります。  
運用をシンプルにするなら、最初は `.env` の `POSTGRES_USER` をアプリ用として使い、必要になったら権限分離してもよいです。

---

# 11. タイムゾーン設定

Compose 側：

```yaml
environment:
  TZ: Asia/Tokyo
  PGTZ: Asia/Tokyo
```

DB 側で確認：

```bash
docker compose exec postgres psql -U app_user -d app_db -c "SHOW timezone;"
```

必要なら `postgresql.conf` で明示する方法もありますが、まずは `PGTZ` で十分なケースが多いです。

---

# 12. パフォーマンス用の最低限設定

PostgreSQL は共有メモリを使うため、Docker では以下を入れておくとよいです。

```yaml
shm_size: 256mb
```

大量のクエリや重い集計をする場合は増やします。

```yaml
shm_size: 1gb
```

ただし、本格的なチューニングはメモリ量、接続数、ワークロード次第です。

---

# 13. ヘルスチェックを入れる

アプリコンテナと一緒に使う場合、PostgreSQL の起動完了を判定できます。

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5
```

アプリ側で依存関係を書く場合：

```yaml
depends_on:
  postgres:
    condition: service_healthy
```

例：

```yaml
services:
  app:
    image: your-app-image
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      DATABASE_URL: postgres://app_user:password@postgres:5432/app_db
```

---

# 14. ログ肥大化対策

Docker のログが増え続けることがあります。  
Compose 側で制限できます。

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

追加すると：

```yaml
services:
  postgres:
    image: postgres:16
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

---

# 15. セキュリティの基本

最低限やること：

- 強いパスワードを使う
- `.env` を Git に入れない
- `ports` は必要な場合だけ公開
- 公開するなら `127.0.0.1` に限定する
- 外部接続が必要なら VPN や SSH トンネルを使う
- Ubuntu の firewall を確認する
- 定期バックアップを取る
- `postgres:latest` を使わない
- 不要な拡張機能を入れない

Ubuntu で UFW を使う場合：

```bash
sudo ufw status
```

外部公開するなら、むやみに以下をしない方がいいです。

```bash
sudo ufw allow 5432
```

---

# 16. 起動・停止・確認コマンド

```bash
cd /opt/postgres
docker compose up -d
```

状態確認：

```bash
docker compose ps
```

ログ確認：

```bash
docker compose logs -f postgres
```

DB 接続：

```bash
docker compose exec postgres psql -U app_user -d app_db
```

停止：

```bash
docker compose down
```

停止してデータも削除：

```bash
docker compose down -v
```

`down -v` は named volume も消すので注意してください。

---

# 17. 本番寄りの完成例

```yaml
services:
  postgres:
    image: postgres:18.3
    restart: unless-stopped

    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      TZ: Asia/Tokyo
      PGTZ: Asia/Tokyo

    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d:ro

    ports:
      - "127.0.0.1:5432:5432"

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

    shm_size: 256mb

    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

    networks:
      - db_net

volumes:
  postgres_data:

networks:
  db_net:
    driver: bridge
```

`.env`：

```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=please_use_a_long_random_password
```

---

# 結論

Docker Compose で PostgreSQL を構築するなら、基本はこれです。

- `compose.yaml` は `/opt/postgres/` などに置く
- DB データは named volume に保存する
- `postgres:latest` は使わない
- `.env` でパスワード管理する
- `ports` は `127.0.0.1` に限定する
- `healthcheck` を入れる
- バックアップ手順を最初に作る
- `docker compose down -v` は慎重に使う

最小構成ならこれで十分です。

```yaml
services:
  postgres:
    image: postgres:18.3
    restart: unless-stopped
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: change_me
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5432:5432"

volumes:
  postgres_data:
```

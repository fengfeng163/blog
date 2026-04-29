---
title: PostgreSQL generate_subscriptsの使い方
date: 2026-04-29
description: PostgreSQL generate_subscripts の使い方
tags: 
    - PostgreSQL
categories:
    - PostgreSQL
---
# PostgreSQL `generate_subscripts` の使い方

`generate_subscripts` は配列の添字(インデックス)を生成する関数で、主に配列をループ処理するために使われます。

## 関数のシグネチャ

```sql
generate_subscripts(array anyarray, dim int) → setof int
generate_subscripts(array anyarray, dim int, reverse boolean) → setof int
```

- `array`: 対象の配列
- `dim`: 配列の次元(1 から始まる)
- `reverse`: 逆順で添字を生成するかどうか(省略可)

## 基本的な使い方

### 1. 一次元配列の添字を生成

```sql
SELECT generate_subscripts(ARRAY['a','b','c','d'], 1) AS idx;
```

出力:

```
 idx
-----
   1
   2
   3
   4
```

### 2. 配列の値と組み合わせて要素を取得

```sql
SELECT i, arr[i] AS value
FROM (SELECT ARRAY['りんご','バナナ','みかん'] AS arr) t,
     generate_subscripts(arr, 1) AS i;
```

出力:

```
 i |  value
---+--------
 1 | りんご
 2 | バナナ
 3 | みかん
```

### 3. 逆順で取得

```sql
SELECT generate_subscripts(ARRAY[10,20,30], 1, true) AS idx;
-- 出力: 3, 2, 1
```

### 4. 二次元配列の処理

```sql
SELECT i, j, arr[i][j] AS val
FROM (SELECT ARRAY[[1,2,3],[4,5,6]] AS arr) t,
     generate_subscripts(arr, 1) AS i,
     generate_subscripts(arr, 2) AS j;
```

出力:

```
 i | j | val
---+---+-----
 1 | 1 |   1
 1 | 2 |   2
 1 | 3 |   3
 2 | 1 |   4
 2 | 2 |   5
 2 | 3 |   6
```

## 実用例

### ケース 1: テーブルの配列カラムを展開する

`tags_table(id int, tags text[])` というテーブルがあるとします:

```sql
SELECT id, i AS position, tags[i] AS tag
FROM tags_table, generate_subscripts(tags, 1) AS i;
```

### ケース 2: 添字付きで条件検索

```sql
SELECT i, arr[i]
FROM (SELECT ARRAY['x','y','z'] AS arr) t,
     generate_subscripts(arr, 1) AS i
WHERE arr[i] = 'y';
```

## `unnest` との違い

- `unnest(arr)`: 要素のみを返し、添字は返さない。
- `unnest(arr) WITH ORDINALITY`: 要素と連番(1 から始まる順序番号)を返す。
- `generate_subscripts`: 実際の配列の添字を返す。PostgreSQL の配列は `arr[0:2]` のように 1 以外の添字から始められるため、より正確。

例:

```sql
SELECT generate_subscripts('[5:7]={a,b,c}'::text[], 1);
-- 出力: 5, 6, 7
```

一方、`unnest WITH ORDINALITY` ではこの場合でも 1, 2, 3 が返されます。

## まとめ

| 用法 | 説明 |
|------|------|
| `generate_subscripts(arr, 1)` | 一次元配列の添字を生成 |
| `generate_subscripts(arr, 1, true)` | 逆順で生成 |
| `generate_subscripts(arr, 2)` | 二次元配列の 2 次元目の添字 |

要素の値だけが必要なら `unnest` が簡潔ですが、**添字が必要な場合(特に非標準の添字を扱う場合)は `generate_subscripts` を使う**のが適切です。
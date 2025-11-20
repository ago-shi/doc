## kvsエンジン
#### kvsに登録されたデータ一覧
```
$ vault kv list <path>
```
#### kvsへのkey/value登録
```
$ vault kv put <path>/<data name> <KEY1>="<Value1>" <KEY2>="<Value2>"
``` 
#### kvsからkey/value情報を取得
```
$ vault kv get <path>/<data name>
```

#### kvsからkey/valueを削除する
```
$ vault kv metadata <path>/<data name>
```
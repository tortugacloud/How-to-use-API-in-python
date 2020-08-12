# 現在のAPIバージョンv1

### Tortuga Cloudにログインしましょう


```python
import requests

# 私たちはあなたのパスワードを知りたくない、または攻撃者がそれを傍受できるようにしたくない
# 承認前にパスワードをハッシュする

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

成功すると、APIメソッドにアクセスするためのトークンを受け取ります

```python
{
  token: '********************.********************.********************'
}
```

### それでは、アカウント情報を取得しましょう

>このようにして、費用と収入を管理できます

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります（この例では、アカウントの役割はパートナーです）

```python
# アカウントの役割によってオブジェクトが異なる場合があります
# idUserStatusとidUserGroupの意味は、ドキュメントの最後の表で確認できます
{
  id: 9,
  idUserStatus: 1,
  idUserGroup: 2,
  idTariff: 1,
  fullname: 'Name Lastname',
  email: 'enter@your.email',
  promoCode: 'ed40cc80',
  balance: 0
}
```

## ファイルシステムの操作

### ストレージの使用済みスペースに関する情報をリクエストする

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります（この例では、アカウントの役割はパートナーです）

```python
# アカウントの役割によってオブジェクトが異なる場合があります
# 測定単位-メガバイト
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### ルートディレクトリの構造に関する情報を要求する

> ルートディレクトリの内容を取得するには、IDの代わりに0を渡します
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

成功すると、アカウントに関する情報を含むJSONオブジェクトを受け取ります

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### ディレクトリを作成

> idParentは現在のidFolderと等しい

```python

r = requests.post('https://tortugacloud.com/api/v1/file-system/folder',
data={
    "idParent": 8,
    "name": "New folder"
},
headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

成功すると、応答を受け取ります

```text
Created
```

また、ルート構造のリクエストを繰り返すと

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# 結果は次のようになります

{
  idFolder: 8,
  folders: [
    {
      id: 9,
      idParent: 8,
      name: 'New folder',
      bucketName: '******************'
    }
  ],
  files: []
}

```

###ディレクトリを削除する

> **注意、ディレクトリの削除は再帰的に実行されます**

```python

r = requests.delete('https://tortugacloud.com/api/v1/file-system/folder',
data={
    "idFolder": 9,
},
headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

> 削除が成功すると、空の応答本文が返されます

```text

```

### ディレクトリの名前を変更する

>ディレクトリIDをidParentと混同しないでください!!
> idParentは、ファイルマネージャーが機能するために必要です

```python
{
...
  folders: [
    {
      id: 10,
      idParent: 8,
      name: 'New folder',
      bucketName: '****************'
    },
    ...
  ],
...
}
```

>たとえば、「新しいフォルダ」の名前をID 10に変更します。

```python

r = requests.put('https://tortugacloud.com/api/v1/file-system/folder',
data={
    "idFolder": 10,
    "name": 'test'
},
headers={
    "Authorization": "JWT " + token
})
print(r.text)

```
>ルートディレクトリに関する情報を呼び出した結果、

```python
{
...
  folders: [
    {
      id: 10,
      idParent: 8,
      name: 'test',
      bucketName: '****************'
    },
    ...
  ],
...
}
```

### ファイルのアップロード

> FormData実装を使用してファイルをアップロードする

```python
import requests
from requests_toolbelt.multipart.encoder import MultipartEncoder

multipart_data = MultipartEncoder(
    fields={
            'files': ('__init__.py', open('__init__.py', 'rb'), 'text/plain'),
           }
    )

r = requests.post('https://tortugacloud.com/api/v1/file-system/file/8',
headers={
    "Authorization": "JWT " + token,
    "Content-Type": multipart_data.content_type,
},
data=multipart_data
)
print(r.text)

```

> idが8のディレクトリに関する情報を呼び出した結果、
```python
{
...
    folders: [...],
    files: [
      {
        id: 1,
        name: '__init__.py',
        sizeInKB: 1.108,
        FolderId: 8,
        objectName: '******'
      }
    ]
}
```

###ファイルの名前を変更する

> ロードした__init__.pyの名前をtest.pyに変更しましょう
> **ファイルの名前には拡張子が含まれており、ファイル名を変更してもMIMEタイプは変更されません**

```python
r = requests.put('https://tortugacloud.com/api/v1/file-system/file',
headers={
    "Authorization": "JWT " + token
}, data={
    "idFile": 21,
    "name": "test.py"
})
print(r.text)
...
# 実行結果
{ status: 200, data: 'OK' }

```

＃＃＃ ダウンロードファイル

>ファイルをダウンロードするには、次の代わりにファイルのobjectNameを渡します：link
> https://tortugacloud.com/api/v1/file-system/file/:link
>ファイルへのリンクを短くするには、[リンク短縮サービス]（https://tortugacloud.com/shorter）を使用します

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/file/***',
headers={
    "Authorization": "JWT " + token
})

print({
    "status": r.status_code,
    "filename": r.headers['content-disposition'],
    "data": r.content
})
# サーバー応答の例

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'ファイルの内容はデモ用に隠されています',
}
```

### ファイルを削除する

> ファイルを削除するには、ファイルのIDとそれが置かれているフォルダーのIDが必要です

```python
# test.pyを削除しましょう

{
...
    folders: [...],
    files: [
      {
        id: 1,
        name: 'test.py',
        sizeInKB: 1.138,
        FolderId: 8,
        objectName: '******'
      }
    ]
}

# これを行うには、リクエストを実行します

r = requests.delete('https://tortugacloud.com/api/v1/file-system/file',
headers={
    "Authorization": "JWT " + token
},
data= {
        "idFile": 1,
        "idFolder": 8
    }
                    )
print(r.content)

```

> サーバーの応答

```python
'{ status: 200, data: 'OK' }'
```

### idUserStatusとは

| idUserStatus | 値|
| ：-----------：| ----------------- |
| 1 |アクティブなアカウント|
| 2 |ブロックされたアカウント|



### idUserGroupとは

| idUserGroup | アカウントの種類|
| ：-----------：| ----------------- |
| 1 |クライアントアカウント|
| 2 |アフィリエイトアカウント|
| 3 |メンバーシップアカウント（コンテンツ利用者）|

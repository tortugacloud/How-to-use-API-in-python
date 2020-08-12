＃当前API版本v1

###让我们登录Tortuga Cloud


```python
import requests

# 我们不想知道您的密码或让攻击者拦截它
# 因此，请在授权之前对您的密码进行哈希处理

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

如果成功，您将收到一个令牌以访问API方法

```python
{
  token: '********************.********************.********************'
}
```

### 太好了，让我们获取您的帐户信息

> 这样您就可以控制您的支出和收入

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息（在此示例中，帐户角色是合作伙伴）

```python
# 根据您帐户的角色，对象可能有所不同
# 您可以在文档末尾的表格中看到idUserStatus和idUserGroup的含义
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
## 使用文件系统

### 请求有关存储空间中已用空间的信息

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息（在此示例中，帐户角色是合作伙伴）

```python
# 根据您帐户的角色，对象可能有所不同
# 度量单位-兆字节
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### 请求有关根目录结构的信息

> 要获取根目录的内容，请传递值0而不是id
> https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

如果成功，您将收到一个JSON对象，其中包含有关您帐户的信息

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### 创建目录

> idParent 等于当前的 idFolder

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

如果成功，您将收到回复

```text
Created
```

此外，如果重复对根结构的请求

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# 结果如下

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
### 删除目录

> **注意，递归删除目录**

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

> 成功删除后，您将获得一个空的响应正文

```text

```

### 重命名目录

> 不要将目录ID与idParent混淆！
> 文件管理器需要idParent才能工作

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

> 例如，将ID为10的“新文件夹”重命名

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
> 通过调用有关根目录的信息，我们得到

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

###上传文件

>使用FormData实现来加载文件

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

>由于调用有关ID等于8的目录的信息，我们得到
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

### 重命名文件

> 让我们重命名刚刚加载到test.py的__init__.py
> **文件名包含其扩展名，更改它不会更改MIME类型**

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
# Результат выполнения 
{ status: 200, data: 'OK' }

```

### 下载文件

> 要下载文件，请传递文件的objectName而不是：link
> https://tortugacloud.com/api/v1/file-system/file/:link
> 要获得指向该文件的简短链接，请使用我们的[链接缩短器]（https://tortugacloud.com/shorter）


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
# Пример ответа сервера 

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: '隐藏文件内容以进行演示',
}
```

### 删除文件

> 要删除文件，我们需要它的ID和它所在的文件夹的ID

```python
# 让我们删除test.py

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

# 为此，执行请求

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

>服务器响应

```python
'{ status: 200, data: 'OK' }'
```

###什么是idUserStatus

| idUserStatus | 价值|
| ：-----------：| ----------------- |
| 1 |有效帐户|
| 2 |被封锁的帐户|



###什么是idUserGroup

| idUserGroup | 帐户类型|
| ：-----------：| ----------------- |
| 1 |客户帐户|
| 2 |会员帐号|
| 3 |会员帐户（内容消费者）|

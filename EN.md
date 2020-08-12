# Current API version v1

### Let's log in to Tortuga Cloud


```python
import requests

# We do not want to know your password or allow an attacker to intercept it
# So hash your password before authorization

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

If successful, you will receive a token to access the API methods

```python
{
  token: '********************.********************.********************'
}
```

### Great, let's get your account information

> This way you can control your expenses and income

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

If successful, you will receive a JSON object with information about your account (In this example, the account role is partner)

```python
# The object may differ depending on the role of your account
# You can see what idUserStatus and idUserGroup mean in the tables at the end of the document
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

## Working with the file system

### Request information about used space in your storage

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

If successful, you will receive a JSON object with information about your account (In this example, the account role is partner)

```python
# The object may differ depending on the role of your account
# Units of measurement - megabytes
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Request information about the structure of the root directory

> To get the contents of the root directory, pass the value 0 instead of id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

If successful, you will receive a JSON object with information about your account

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Create directory

> idParent equals the current idFolder

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

If successful, you will receive a response

```text
Created
```

Moreover, if you repeat the request for the root structure

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# The result will be as follows

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

### Deleting a directory

> **Attention, deleting a directory is performed recursively**

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

> On successful deletion you will get an empty response body

```text

```

### Renaming a directory

> Do not confuse directory id with idParent !!
> idParent is needed for the file manager to work

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

> For example, rename the 'New Folder' with id 10

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
> As a result of calling information about the root directory, we get

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

### Uploading files

> Use FormData implementation to upload file

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

> As a result of calling information about the directory with id equal to 8, we get
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

### Rename a file

> Let's rename the __init__.py we just loaded to test.py
> **The file name contains its extension, changing it does not change the MIME type**

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

### Download file

> To download a file, pass the objectName of the file instead of: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> For a nice and short link to the file, use our [link shortener] (https://tortugacloud.com/shorter)


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
  data: 'File content is hidden for demonstration',
}
```

### Deleting a file

> To delete a file, we need its id and the id of the folder in which it is located

```python
# Let's remove our test.py

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

# To do this, execute the request

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

> Server response

```python
'{status: 200, data:' OK '}'
```

### What is idUserStatus

| idUserStatus | Value |
| : -----------: | ----------------- |
| 1 | active account |
| 2 | blocked account |



### What is idUserGroup

| idUserGroup | Account type |
| : -----------: | ----------------- |
| 1 | Client Account |
| 2 | Affiliate Account |
| 3 | Membership Account (Content Consumers) |
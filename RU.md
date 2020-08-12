# Текущая версия API v1

### Давайте авторизируемся в Tortuga Cloud


```python
import requests

# Мы не хотим знать ваш пароль или давать возможность злоумышленику перехватить его
# Поэтому захешируйте свой пароль перед авторизацией

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

При успешном выполнении вы получите токен для доступа к методам API

```python
{
  token: '********************.********************.********************'
}
```

### Отлично, давайте получим информацию о вашей учётной записи  

> Так вы сможете контролировать ваши расходы и доходы 

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи (В данном примере роль учётной записи - партнёр)

```python
# Объект может отличаться в зависимости от роли вашей учётной записи 
# Что означают idUserStatus и idUserGroup вы можете в таблицах в конце документа
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

## Работа с файловой системой 

### Запрос информации об израсходованном месте в вашем хранилище

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи (В данном примере роль учётной записи - партнёр)

```python
# Объект может отличаться в зависимости от роли вашей учётной записи 
# Единицы измерения - мегабайты
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Запрос информации о структуре корневой директории 

>  Для получения содержимого корневой директории передайте значение 0 вместо id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

При успешном выполнении вы получите JSON объект с информацией о вашей учётной записи

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Создание директории  

>  idParent равняется текущему idFolder

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

При успешном выполнении вы получите ответ

```text
Created
```

При этом если повторить запрос о корневой структуре 

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# Результат будет следующим 

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

### Удаление директории

> **Внимание, удалении директории выполняется рекурсивно**

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

> При успешном удалении вы получите пустое тело ответа

```text

```

### Переименование директории

> Не путайте id директории с idParent !!  
> idParent нужен для работы файлового менеджера

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

> Для примера переименуем папку 'New Folder' с id 10

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
> В результате вызова информации о корневой директории получим

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

### Загрузка файлов 

> Для загрузки файлом воспользуйтесь реализацией FormData

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

> В результате вызова информации о директории c id равным 8 получим  
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

### Переименование файла 

> Давайте переименуем только что загруженный __init__.py в test.py 
> **Имя файла содержит его расширение, меняя его, вы не меняете MIME-тип**

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

### Скачивание файла

> Для скачивания файла передайте objectName файла вместо :link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Для получения красивой и короткой ссылки на файл воспользуйтесь нашим [укорачивателем ссылок](https://tortugacloud.com/shorter)


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
  data: 'Содержимое файла скрыто для демонстрации',
}
```

### Удаление файла

> Для удаление файла нам понадобится его id и id папки в которой он находится

```python
# Давайте удалим наш test.py 

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

# Для этого выполним запрос

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

> Ответ сервера

```python
'{ status: 200, data: 'OK' }'
```

### Что такое idUserStatus 

| idUserStatus  | Значение        |
| :-----------: |-----------------|
|      1        |активная учётная  запись |
|      2        |заблокированная учётная запись |



### Что такое idUserGroup

| idUserGroup  | Тип учётной записи        |
| :-----------: |-----------------|
|      1        |Клиентская учётная запись |
|      2        |Партнёрская учётная запись |
|      3        |Членская учётная запись (Потребители контента) |

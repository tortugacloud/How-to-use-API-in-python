# Поточна версія API v1

### Давайте авторізіруемся в Tortuga Cloud


```python
import requests

# Ми не хочемо знати ваш пароль або давати можливість зловмисник перехопити його
# Тому захешіруйте свій пароль перед авторизацією

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

При успішному виконанні ви отримаєте токен для доступу до методів API

```python
{
  token: '********************.********************.********************'
}
```

### Відмінно, давайте отримаємо інформацію про ваш обліковий запис

> Так ви зможете контролювати ваші витрати і доходи

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис (В даному прикладі роль облікового запису - партнер)

```python
# Об'єкт може відрізнятися в залежності від ролі вашого облікового запису
# Що означають idUserStatus і idUserGroup ви можете в таблицях в кінці документа
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

## Робота з файловою системою

### Запит інформації про витрачені місці в вашому сховищі

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис (В даному прикладі роль облікового запису - партнер)

```python
# Об'єкт може відрізнятися в залежності від ролі вашого облікового запису
# Одиниці виміру - мегабайти
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Запит інформації про структуру кореневої директорії

> Для отримання вмісту кореневої директорії передайте значення 0 замість id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

При успішному виконанні ви отримаєте JSON об'єкт з інформацією про ваш обліковий запис

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Створення директорії

> IdParent дорівнює поточному idFolder

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

При успішному виконанні ви отримаєте відповідь

```text
Created
```

При цьому якщо повторити запит про кореневої структурі

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# Результат буде наступним

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

### Видалення директорії

> **Увага, видаленні директорії виконується рекурсивно**

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

> При успішному видаленні ви отримаєте порожній тіло відповіді

```text

```

### Перейменування директорії

> Не плутайте id директорії з idParent !!
> IdParent потрібен для роботи файлового менеджера

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

> Для прикладу перейменуємо папку 'New Folder' з id 10

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
> В результаті виклику інформації про кореневій директорії отримаємо

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

### Завантаження файлів

> Для завантаження файлом скористайтеся реалізацією FormData

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

> В результаті виклику інформації про директорії c id рівним 8 отримаємо
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

### Перейменування файлу

> Давайте перейменуємо тільки що завантажений __init__.py в test.py
> **Файл містить його розширення, змінюючи його, ви не міняєте MIME-тип**

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
# Результат виконання
{ status: 200, data: 'OK' }

```

### Завантаження файлу

> Для скачування файлу передайте objectName файлу замість: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Для отримання красивою і короткою посилання на файл скористайтеся нашим [укорачівателем посилань](https://tortugacloud.com/shorter)


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
# Приклад відповіді сервера

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'Вміст файлу приховано для демонстрації',
}
```

### Видалення файлу

> Для видалення файлу нам знадобиться його id і id папки в якій він знаходиться

```python
# Давайте видалимо наш test.py

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

# Для цього виконаємо запит

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

> Відповідь сервера

```python
'{ status: 200, data: 'OK' }'
```

### Що таке idUserStatus

| idUserStatus | значення |
| : -----------: | ----------------- |
| 1 | активна обліковий запис |
| 2 | заблокована обліковий запис |



### Що таке idUserGroup

| idUserGroup | Тип облікового запису |
| : -----------: | ----------------- |
| 1 | Клієнтська обліковий запис |
| 2 | Партнерська обліковий запис |
| 3 | Членська обліковий запис (Споживачі контенту) |

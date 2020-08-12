# إصدار API الحالي v1

### دعنا نسجل الدخول إلى Tortuga Cloud


```python
import requests

# لا نريد معرفة كلمة المرور الخاصة بك أو السماح للمهاجمين باعتراضها
# لذلك ، قم بتجزئة كلمة المرور الخاصة بك قبل الإذن

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

إذا نجحت ، ستتلقى رمزًا مميزًا للوصول إلى طرق API

```python
{
  token: '********************.********************.********************'
}
```

### رائع ، دعنا نحصل على معلومات حسابك

> بهذه الطريقة يمكنك التحكم في نفقاتك ودخلك

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

إذا نجحت ، فستتلقى كائن JSON به معلومات حول حسابك (في هذا المثال ، دور الحساب هو الشريك)

```python
# قد يختلف الكائن حسب دور حسابك
# يمكنك أن ترى ما تعنيه idUserStatus و idUserGroup في الجداول الموجودة في نهاية المستند
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

## العمل مع نظام الملفات

### اطلب معلومات حول المساحة المستخدمة في التخزين لديك

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

إذا نجحت ، فستتلقى كائن JSON به معلومات حول حسابك (في هذا المثال ، دور الحساب هو الشريك)

```python
# قد يختلف الكائن حسب دور حسابك
# وحدات القياس - ميغا بايت
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### طلب معلومات حول بنية الدليل الجذر

> مرر 0 بدلاً من id للحصول على محتويات الدليل الجذر
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

إذا نجحت ، فسوف تتلقى كائن JSON مع معلومات حول حسابك

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### إنشاء دليل

> idParent يساوي idFolder الحالي

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

إذا نجحت ، ستتلقى ردًا

```text
Created
```

علاوة على ذلك ، إذا كررت طلب بنية الجذر

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# ستكون النتيجة على النحو التالي

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

### حذف دليل

> **تنبيه ، يتم حذف دليل بشكل متكرر**

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

> عند الحذف بنجاح ، ستحصل على نص استجابة فارغ

```text

```

### إعادة تسمية دليل

> لا تخلط بين معرف الدليل و idParent !!
> يلزم وجود idParent لكي يعمل مدير الملفات

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

> على سبيل المثال ، أعد تسمية "مجلد جديد" بالمعرف 10

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
> نتيجة لاستدعاء معلومات حول الدليل الجذر ، نحصل على

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

### تحميل الملفات

> استخدام تطبيق FormData لتحميل الملف

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

> نتيجة لاستدعاء معلومات حول الدليل مع معرف يساوي 8 ، نحصل على
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

### إعادة تسمية ملف

> دعنا نعيد تسمية __init__.py الذي قمنا بتحميله للتو إلى test.py
> ** اسم الملف يحتوي على امتداده ، وتغييره لا يغير نوع MIME **

```python
r = requests.put('https://tortugacloud.com/api/v1/file-system/file',
headers={
    "Authorization": "JWT " + token
}, data={
    "idFile": 21,
    "name": "test.py"
})
print(r.content)
...
# نتيجة التنفيذ
{ status: 200, data: 'OK' }

```

### تحميل الملف

> لتنزيل ملف ، قم بتمرير اسم الكائن للملف بدلاً من: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> للحصول على رابط لطيف ومختصر للملف ، استخدم [link shortener] (https://tortugacloud.com/shorter)


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
  data: 'محتوى الملف مخفي للشرح',
}
```

### حذف ملف

> لحذف ملف ، نحتاج إلى معرفه ومعرف المجلد الذي يوجد فيه

```python
# دعونا نحذف test.py
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

# للقيام بذلك ، قم بتنفيذ الطلب

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

> استجابة الخادم

```python
'{ status: 200, data: 'OK' }'
```

### ما هو idUserStatus

| idUserStatus | القيمة |
| : -----------: | ----------------- |
| 1 | حساب نشط |
| 2 | حساب مغلق |



### ما هو idUserGroup

| idUserGroup | نوع الحساب |
| : -----------: | ----------------- |
| 1 | حساب العميل |
| 2 | حساب تابع |
| 3 | حساب العضوية (مستهلكو المحتوى) |

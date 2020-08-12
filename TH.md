# API ปัจจุบันเวอร์ชัน v1.0

### เข้าสู่ระบบ Tortuga Cloud กันเถอะ


```python
import requests

# เราไม่ต้องการทราบรหัสผ่านของคุณหรืออนุญาตให้ผู้โจมตีสกัดกั้น
# ดังนั้นให้แฮชรหัสผ่านของคุณก่อนการอนุญาต

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

หากสำเร็จคุณจะได้รับโทเค็นเพื่อเข้าถึงเมธอด API

```python
{
  token: '********************.********************.********************'
}
```

### เยี่ยมมากขอข้อมูลบัญชีของคุณ

> วิธีนี้คุณสามารถควบคุมค่าใช้จ่ายและรายได้ของคุณ

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ (ในตัวอย่างนี้บทบาทบัญชีคือคู่ค้า)

```python
# วัตถุอาจแตกต่างกันไปขึ้นอยู่กับบทบาทของบัญชีของคุณ
# คุณสามารถดูความหมายของ idUserStatus และ idUserGroup ได้ในตารางท้ายเอกสาร
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

## การทำงานกับระบบไฟล์

### ขอข้อมูลเกี่ยวกับพื้นที่ที่ใช้ในการจัดเก็บของคุณ

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ (ในตัวอย่างนี้บทบาทบัญชีคือคู่ค้า)

```python
# วัตถุอาจแตกต่างกันไปขึ้นอยู่กับบทบาทของบัญชีของคุณ
# หน่วยวัด - เมกะไบต์
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### ขอข้อมูลเกี่ยวกับโครงสร้างของไดเรกทอรีราก

> ในการรับเนื้อหาของไดเร็กทอรีรูทให้ส่งค่า 0 แทน id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

หากสำเร็จคุณจะได้รับออบเจ็กต์ JSON พร้อมข้อมูลเกี่ยวกับบัญชีของคุณ

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### สร้างไดเร็กทอรี

> idParent เท่ากับ idFolder ปัจจุบัน

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

หากประสบความสำเร็จคุณจะได้รับการตอบสนอง

```text
Created
```

ยิ่งไปกว่านั้นหากคุณทำซ้ำการร้องขอโครงสร้างรูท

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# ผลลัพธ์จะเป็นดังนี้

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

### การลบไดเร็กทอรี

> **ความสนใจการลบไดเรกทอรีจะดำเนินการซ้ำ**

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

> เมื่อลบสำเร็จคุณจะได้รับเนื้อหาตอบกลับที่ว่างเปล่า

```text

```

### การเปลี่ยนชื่อไดเรกทอรี

> อย่าสับสนระหว่าง id ไดเรกทอรีกับ idParent !!
> idParent จำเป็นสำหรับตัวจัดการไฟล์ในการทำงาน

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

> ตัวอย่างเช่นเปลี่ยนชื่อ 'โฟลเดอร์ใหม่' ด้วย id 10

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
> จากการเรียกข้อมูลเกี่ยวกับไดเร็กทอรีรากเราได้รับ

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

### การอัปโหลดไฟล์

> ใช้การใช้งาน FormData เพื่ออัปโหลดไฟล์

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

> เนื่องจากการเรียกข้อมูลเกี่ยวกับไดเร็กทอรีที่มี id เท่ากับ 8 เราจึงได้รับ
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

### เปลี่ยนชื่อไฟล์

> มาเปลี่ยนชื่อ __init__.py ที่เราเพิ่งโหลดไปที่ test.py
> ** ชื่อไฟล์มีนามสกุลการเปลี่ยนไม่เปลี่ยนประเภท MIME **

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
# ผลการดำเนินการ
{ status: 200, data: 'OK' }

```

### ดาวน์โหลดไฟล์

> ในการดาวน์โหลดไฟล์ให้ส่ง objectName ของไฟล์แทน: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> สำหรับลิงก์ที่ดีและสั้นไปยังไฟล์ให้ใช้ [link shortener] ของเรา (https://tortugacloud.com/shorter)


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
# ตัวอย่างการตอบสนองของเซิร์ฟเวอร์

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'เนื้อหาไฟล์ถูกซ่อนไว้เพื่อการสาธิต',
}
```

### การลบไฟล์

> ในการลบไฟล์เราต้องใช้ id และ id ของโฟลเดอร์ที่ไฟล์นั้นอยู่

```python
# มาลบ test.py ของเรา

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

# ในการดำเนินการนี้ให้ดำเนินการตามคำขอ

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

> การตอบสนองของเซิร์ฟเวอร์

```python
'{ status: 200, data: 'OK' }'
```

### idUserStatus คืออะไร

| idUserStatus | มูลค่า |
| : -----------: | ----------------- |
| 1 | บัญชีที่ใช้งานอยู่ |
| 2 | บัญชีที่ถูกบล็อก |



### idUserGroup คืออะไร

| idUserGroup | ประเภทบัญชี |
| : -----------: | ----------------- |
| 1 | บัญชีลูกค้า |
| 2 | บัญชีพันธมิตร |
| 3 | บัญชีสมาชิก (ผู้บริโภคเนื้อหา) |

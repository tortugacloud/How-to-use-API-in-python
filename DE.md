# Aktuelle API-Version v1

### Melden wir uns bei Tortuga Cloud an


```python
import requests

# Wir möchten Ihr Passwort nicht kennen oder einem Angreifer erlauben, es abzufangen
# Hashing also dein Passwort vor der Autorisierung

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

Bei Erfolg erhalten Sie ein Token für den Zugriff auf die API-Methoden

```python
{
  token: '********************.********************.********************'
}
```

### Großartig, lassen Sie uns Ihre Kontoinformationen abrufen

> Auf diese Weise können Sie Ihre Ausgaben und Einnahmen kontrollieren

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto (in diesem Beispiel ist die Kontorolle Partner).

```python
# Das Objekt kann je nach Rolle Ihres Kontos unterschiedlich sein
# Sie können in den Tabellen am Ende des Dokuments sehen, was idUserStatus und idUserGroup bedeuten
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

## Arbeiten mit dem Dateisystem

### Fordern Sie Informationen zum verwendeten Speicherplatz in Ihrem Speicher an

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto (in diesem Beispiel ist die Kontorolle Partner).

```python
# Das Objekt kann je nach Rolle Ihres Kontos unterschiedlich sein
# Maßeinheiten - Megabyte
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Informationen zur Struktur des Stammverzeichnisses anfordern

> Um den Inhalt des Stammverzeichnisses abzurufen, übergeben Sie den Wert 0 anstelle von id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Bei Erfolg erhalten Sie ein JSON-Objekt mit Informationen zu Ihrem Konto

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Verzeichnis erstellen

> idParent entspricht dem aktuellen idFolder

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

Bei Erfolg erhalten Sie eine Antwort

```text
Created
```

Darüber hinaus, wenn Sie die Anforderung für die Stammstruktur wiederholen

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# Das Ergebnis ist wie folgt

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

### Ein Verzeichnis löschen

> **Achtung, das Löschen des Verzeichnisses erfolgt rekursiv**

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

> Nach erfolgreichem Löschen erhalten Sie einen leeren Antworttext

```text

```

### Umbenennen eines Verzeichnisses

> Verwechseln Sie die Verzeichnis-ID nicht mit idParent !!
> idParent wird benötigt, damit der Dateimanager funktioniert

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

> Benennen Sie beispielsweise den 'Neuen Ordner' mit der ID 10 um

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
> Als Ergebnis des Aufrufs von Informationen über das Stammverzeichnis erhalten wir

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

### Dateien hochladen

> Verwenden Sie die FormData-Implementierung, um eine Datei hochzuladen

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

> Als Ergebnis des Aufrufs von Informationen über das Verzeichnis mit einer ID von 8 erhalten wir
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

### Benennen Sie eine Datei um

> Benennen wir die gerade geladene __init__.py in test.py um
> **Der Dateiname enthält seine Erweiterung. Durch Ändern wird der MIME-Typ nicht geändert.**

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
# Ausführungsergebnis
{ status: 200, data: 'OK' }

```

### Download-Datei

> Um eine Datei herunterzuladen, übergeben Sie den Objektnamen der Datei anstelle von: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Verwenden Sie für einen schönen und kurzen Link zur Datei unseren [Link Shortener](https://tortugacloud.com/shorter).


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
# Beispiel für eine Serverantwort

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'Der Dateiinhalt wird zur Demonstration ausgeblendet',
}
```

### Eine Datei löschen

> Um eine Datei zu löschen, benötigen wir ihre ID und die ID des Ordners, in dem sie sich befindet

```python
# Entfernen wir unsere test.py

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

# Führen Sie dazu die Anfrage aus

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

> Serverantwort

```python
'{ status: 200, data: 'OK' }'
```

### Was ist idUserStatus?

| idUserStatus | Wert |
| : -----------: | ----------------- |
| 1 | aktives Konto |
| 2 | gesperrtes Konto |



### Was ist idUserGroup?

| idUserGroup | Kontotyp |
| : -----------: | ----------------- |
| 1 | Kundenkonto |
| 2 | Affiliate-Konto |
| 3 | Mitgliedskonto (Inhaltskonsumenten) |

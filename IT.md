# Versione API corrente v1

### Accediamo a Tortuga Cloud


```python
import requests

# Non vogliamo conoscere la tua password o consentire a un utente malintenzionato di intercettarla
# Quindi hash la tua password prima dell'autorizzazione

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

In caso di successo, riceverai un token per accedere ai metodi API

```python
{
  token: '********************.********************.********************'
}
```

### Ottimo, otteniamo le informazioni sul tuo account

> In questo modo puoi controllare le tue spese e le tue entrate

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

In caso di successo, riceverai un oggetto JSON con informazioni sul tuo account (in questo esempio, il ruolo dell'account è partner)

```python
# L'oggetto può differire a seconda del ruolo del tuo account
# Puoi vedere cosa significano idUserStatus e idUserGroup nelle tabelle alla fine del documento
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

## Lavorare con il file system

### Richiedi informazioni sullo spazio utilizzato nel tuo spazio di archiviazione

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

In caso di successo, riceverai un oggetto JSON con informazioni sul tuo account (in questo esempio, il ruolo dell'account è partner)

```python
# L'oggetto può differire a seconda del ruolo del tuo account
# Unità di misura - megabyte
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Richiedi informazioni sulla struttura della directory principale

> Passa 0 invece di id per ottenere il contenuto della directory principale
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

In caso di successo, riceverai un oggetto JSON con le informazioni sul tuo account

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Crea directory

> idParent è uguale a idFolder corrente

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

In caso di esito positivo, riceverai una risposta

```text
Created
```

Inoltre, se ripeti la richiesta per la struttura radice

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# Il risultato sarà il seguente

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

### Eliminazione di una directory

> **Attenzione, l'eliminazione della directory viene eseguita in modo ricorsivo**

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

> In caso di eliminazione riuscita, otterrai un corpo di risposta vuoto

```text

```

### Rinominare una directory

> Non confondere l'id della directory con idParent !!
> idParent è necessario affinché il file manager funzioni

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

> Ad esempio, rinomina la "Nuova cartella" con ID 10

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
> Come risultato della chiamata di informazioni sulla directory root, otteniamo

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

### Caricamento di file

> Usa l'implementazione di FormData per caricare il file

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

> Come risultato della chiamata di informazioni sulla directory con id uguale a 8, otteniamo
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

### Rinomina un file

> Rinominiamo __init__.py che abbiamo appena caricato in test.py
> **Il nome del file contiene la sua estensione, cambiarlo non cambia il tipo MIME**

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
# Risultato dell'esecuzione
{ status: 200, data: 'OK' }

```

### Download file

> Per scaricare un file, passare il objectName del file invece di: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Per un collegamento breve e piacevole al file, usa il nostro [link shortener] (https://tortugacloud.com/shorter)


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
  data: 'Il contenuto del file è nascosto per dimostrazione',
}
```

### Eliminazione di un file

> Per eliminare un file, abbiamo bisogno del suo ID e dell'ID della cartella in cui si trova

```python
# Rimuoviamo il nostro test.py

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

# Per fare ciò, eseguire la richiesta

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

> Risposta del server

```python
'{ status: 200, data: 'OK' }'
```

### Cos'è idUserStatus

| idUserStatus | Valore |
| : -----------: | ----------------- |
| 1 | account attivo |
| 2 | account bloccato |



### Cos'è idUserGroup

| idUserGroup | Tipo di conto |
| : -----------: | ----------------- |
| 1 | Account cliente |
| 2 | Account affiliato |
| 3 | Account di iscrizione (consumatori di contenuti) |

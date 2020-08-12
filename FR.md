# Version actuelle de l'API v1

### Connectons-nous à Tortuga Cloud


```python
import requests

# Nous ne voulons pas connaître votre mot de passe ou permettre à un attaquant de l'intercepter
# Alors hachez votre mot de passe avant l'autorisation

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

En cas de succès, vous recevrez un jeton pour accéder aux méthodes de l'API

```python
{
  token: '********************.********************.********************'
}
```

### Super, obtenons les informations de votre compte

> De cette façon, vous pouvez contrôler vos dépenses et vos revenus

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte (dans cet exemple, le rôle du compte est partenaire)

```python
# L'objet peut différer selon le rôle de votre compte
# Vous pouvez voir ce que signifient idUserStatus et idUserGroup dans les tableaux à la fin du document
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

## Travailler avec le système de fichiers

### Demander des informations sur l'espace utilisé dans votre stockage

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte (dans cet exemple, le rôle du compte est partenaire)

```python
# L'objet peut différer selon le rôle de votre compte
# Unités de mesure - mégaoctets
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Demander des informations sur la structure du répertoire racine

> Pour obtenir le contenu du répertoire racine, passez la valeur 0 au lieu de id
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

En cas de succès, vous recevrez un objet JSON avec des informations sur votre compte

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Créer un répertoire

> idParent est égal à l'idFolder actuel

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

En cas de succès, vous recevrez une réponse

```text
Created
```

De plus, si vous répétez la demande de la structure racine

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# Le résultat sera le suivant

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

### Suppression d'un répertoire

> **Attention, la suppression d'un répertoire est effectuée de manière récursive**

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

> En cas de suppression réussie, vous obtiendrez un corps de réponse vide

```text

```

### Renommer un répertoire

> Ne confondez pas l'identifiant de répertoire avec idParent !!
> idParent est nécessaire pour que le gestionnaire de fichiers fonctionne

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

> Par exemple, renommez le 'Nouveau dossier' avec l'ID 10

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
> À la suite de l'appel d'informations sur le répertoire racine, nous obtenons

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

### Téléchargement de fichiers

> Utilisez l'implémentation FormData pour télécharger le fichier

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

> À la suite de l'appel d'informations sur le répertoire avec un id égal à 8, nous obtenons
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

### Renommer un fichier

> Renommons le __init__.py que nous venons de charger en test.py
> **Le nom du fichier contient son extension, sa modification ne change pas le type MIME**

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
# Résultat de l'exécution
{ status: 200, data: 'OK' }

```
### Télécharger un fichier

> Pour télécharger un fichier, passez le nom d'objet du fichier au lieu de: lien
> https://tortugacloud.com/api/v1/file-system/file/:link
> Pour un lien agréable et court vers le fichier, utilisez notre [raccourcisseur de lien](https://tortugacloud.com/shorter)


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
  data: 'Le contenu du fichier est masqué pour la démonstration',
}
```

### Suppression d'un fichier

> Pour supprimer un fichier, nous avons besoin de son identifiant et de l'identifiant du dossier dans lequel il se trouve

```python
# Supprimons notre test.py

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

# Pour ce faire, exécutez la requête

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

> Réponse du serveur

```python
'{ status: 200, data: 'OK' }'
```

### Qu'est-ce que idUserStatus

| idUserStatus | Valeur |
| : -----------: | ----------------- |
| 1 | compte actif |
| 2 | compte bloqué |



### Qu'est-ce que idUserGroup

| idUserGroup | Type de compte |
| : -----------: | ----------------- |
| 1 | Compte client |
| 2 | Compte affilié |
| 3 | Compte de membre (consommateurs de contenu) |

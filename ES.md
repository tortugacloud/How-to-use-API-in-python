# Versión actual de API v1

### Inicie sesión en Tortuga Cloud


```python
import requests

# No queremos saber su contraseña ni permitir que un atacante la intercepte
# Así que hash tu contraseña antes de la autorización

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

Si tiene éxito, recibirá un token para acceder a los métodos de la API

```python
{
  token: '********************.********************.********************'
}
```

### Genial, obtengamos la información de tu cuenta

> De esta forma puedes controlar tus gastos e ingresos

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta (en este ejemplo, el rol de la cuenta es socio)

```python
# El objeto puede diferir según la función de su cuenta
# Puede ver qué significan idUserStatus e idUserGroup en las tablas al final del documento
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

## Trabajando con el sistema de archivos

### Solicita información sobre el espacio utilizado en tu almacenamiento

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta (en este ejemplo, el rol de la cuenta es socio)

```python
# El objeto puede diferir según la función de su cuenta
# Unidades de medida - megabytes
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Solicita información sobre la estructura del directorio raíz

> Pase 0 en lugar de id para obtener el contenido del directorio raíz
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Si tiene éxito, recibirá un objeto JSON con información sobre su cuenta

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Crear el directorio

> idParent es igual al idFolder actual

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

Si tiene éxito, recibirá una respuesta

```text
Created
```

Además, si repite la solicitud de la estructura raíz

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# El resultado será el siguiente

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

### Eliminando un directorio

> **Atención, la eliminación de un directorio se realiza de forma recursiva**

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

> Si se elimina correctamente, obtendrá un cuerpo de respuesta vacío

```text

```

### Cambiar el nombre de un directorio

> ¡No confunda la identificación del directorio con idParent!
> idParent es necesario para que el administrador de archivos funcione

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

> Por ejemplo, cambie el nombre de la 'Nueva carpeta' con id 10

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
> Como resultado de llamar a información sobre el directorio raíz, obtenemos

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

### Subiendo archivos

> Utilice la implementación de FormData para cargar el archivo

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

> Como resultado de llamar a información sobre el directorio con id igual a 8, obtenemos
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

### Cambiar el nombre de un archivo

> Cambiemos el nombre del __init__.py que acabamos de cargar a test.py
> **El nombre del archivo contiene su extensión, cambiarlo no cambia el tipo MIME**

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
# Resultado de ejecución
{ status: 200, data: 'OK' }

```

### Descargar archivo

> Para descargar un archivo, pase el objectName del archivo en lugar de: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Para obtener un enlace bonito y breve al archivo, utilice nuestro [acortador de enlaces](https://tortugacloud.com/shorter)


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
# Ejemplo de respuesta del servidor

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'El contenido del archivo está oculto para demostración',
}
```

### Eliminando un archivo

> Para eliminar un archivo, necesitamos su id y el id de la carpeta en la que se encuentra

```python
# Eliminemos nuestro test.py

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

# Para hacer esto, ejecute la solicitud

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

> Respuesta del servidor

```python
'{ status: 200, data: 'OK' }'
```

### ¿Qué es idUserStatus?

| idUserStatus | Valor |
| : -----------: | ----------------- |
| 1 | cuenta activa |
| 2 | cuenta bloqueada |



### Qué es idUserGroup

| idUserGroup | Tipo de cuenta |
| : -----------: | ----------------- |
| 1 | Cuenta de cliente |
| 2 | Cuenta de afiliado |
| 3 | Cuenta de membresía (consumidores de contenido) |

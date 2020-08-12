# Versão atual da API v1

### Vamos entrar no Tortuga Cloud


```python
import requests

# Não queremos saber sua senha ou permitir que um invasor a intercepte
# Então, hash sua senha antes da autorização

user = {
    "email": "enter@your.email",
    "password": "25d55ad283aa400af464c76d713c07ad"
}

r = requests.post('https://tortugacloud.com/api/v1/user/login', data=user)
print(r.text)
```

Se for bem-sucedido, você receberá um token para acessar os métodos da API

```python
{
  token: '********************.********************.********************'
}
```

### Ótimo, vamos obter as informações da sua conta

> Desta forma, você pode controlar suas despesas e receitas

```python

token = "***.***.***"

r = requests.get('https://tortugacloud.com/api/v1/user/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)
```

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta (neste exemplo, a função da conta é parceiro)

```python
# O objeto pode ser diferente dependendo da função de sua conta
# Você pode ver o que idUserStatus e idUserGroup significam nas tabelas no final do documento
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

## Trabalhando com o sistema de arquivos

### Solicite informações sobre o espaço usado em seu armazenamento

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/', data=user, headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta (neste exemplo, a função da conta é parceiro)

```python
# O objeto pode ser diferente dependendo da função de sua conta
# Unidades de medida - megabytes
{ 
    used: 0, 
    total: '10240000.000' 
}
```

### Solicitar informações sobre a estrutura do diretório raiz

> Passe 0 em vez de id para obter o conteúdo do diretório raiz
>  https://tortugacloud.com/api/v1/file-system/folder/:id

```python

r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0', headers={
    "Authorization": "JWT " + token
})
print(r.text)

```

Se for bem-sucedido, você receberá um objeto JSON com informações sobre sua conta

```python
{ 
    idFolder: 8, 
    folders: [], 
    files: [] 
}
```

### Criar diretório

> idParent é igual ao idFolder atual

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

Se for bem sucedido, você receberá uma resposta

```text
Created
```

Além disso, se você repetir a solicitação para a estrutura raiz

```python
r = requests.get('https://tortugacloud.com/api/v1/file-system/folder/0',
headers={
    "Authorization": "JWT " + token
})
print(r.text)
# ...
# O resultado será o seguinte

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

### Excluindo um diretório

> **Atenção, a exclusão do diretório é realizada recursivamente**

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

> Na exclusão bem-sucedida, você obterá um corpo de resposta vazio

```text

```

### Renomeando um diretório

> Não confunda o id do diretório com idParent !!
> idParent é necessário para o gerenciador de arquivos funcionar

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

> Por exemplo, renomeie a 'Nova Pasta' com id 10

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
> Como resultado da chamada de informações sobre o diretório raiz, obtemos

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

### Carregando arquivos

> Use a implementação de FormData para fazer o upload do arquivo

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

> Como resultado da chamada de informações sobre o diretório com id igual a 8, obtemos
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

### Renomear um arquivo

> Vamos renomear o __init__.py que acabamos de carregar para test.py
> ** O nome do arquivo contém sua extensão, alterá-lo não altera o tipo MIME **

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
# Resultado de execução
{ status: 200, data: 'OK' }

```

### ⇬ Fazer download do arquivo

> Para baixar um arquivo, passe o nome do objeto do arquivo em vez de: link
> https://tortugacloud.com/api/v1/file-system/file/:link
> Para um link agradável e curto para o arquivo, use nosso [link shortener](https://tortugacloud.com/shorter)


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
# Exemplo de resposta do servidor

{
  status: 200,
  filename: 'attachment; filename="test.py"',
  data: 'O conteúdo do arquivo está oculto para demonstração',
}
```

### Excluindo um arquivo

> Para deletar um arquivo, precisamos do seu id e do id da pasta em que está localizado

```python
# Vamos remover nosso test.py

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

# Para fazer isso, execute a solicitação

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

> Resposta do servidor

```python
'{ status: 200, data: 'OK' }'
```

### O que é idUserStatus

| idUserStatus | Valor |
| : -----------: | ----------------- |
| 1 | conta ativa |
| 2 | conta bloqueada |



### O que é idUserGroup

| idUserGroup | Tipo de conta |
| : -----------: | ----------------- |
| 1 | Conta do cliente |
| 2 | Conta de afiliado |
| 3 | Conta de membro (consumidores de conteúdo) |

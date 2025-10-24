# ddb_client - Cliente DynamoDB

Documentación completa del módulo `ddb_client` para operaciones con AWS DynamoDB.

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Arquitectura](#arquitectura)
- [Configuración](#configuración)
- [Módulo Searches](#módulo-searches)
- [Módulo Updates](#módulo-updates)
- [Helpers](#helpers)
- [Mejores Prácticas](#mejores-prácticas)
- [Solución de Problemas](#solución-de-problemas)

## 🎯 Introducción

El módulo `ddb_client` proporciona una interfaz de alto nivel para interactuar con AWS DynamoDB, abstrayendo la complejidad del SDK de boto3 y proporcionando funcionalidades adicionales como:

- ✅ Validación automática de esquemas
- ✅ Manejo de palabras reservadas de DynamoDB
- ✅ Operaciones batch optimizadas
- ✅ Paginación automática
- ✅ Helpers de comparación avanzados
- ✅ Logging estructurado

### Ventajas sobre boto3 directo

| Característica | boto3 | ddb_client |
|----------------|-------|------------|
| Validación de esquema | ❌ | ✅ |
| Manejo de palabras reservadas | Manual | Automático |
| Operaciones batch | Manual | Simplificado |
| Paginación | Manual | Automático |
| Helpers de comparación | ❌ | ✅ |
| Type hints | Parcial | Completo |

## 🏗️ Arquitectura

```
ddb_client/
├── __init__.py         # Exports principales
├── config.py           # Configuración global
├── constants.py        # Constantes (palabras reservadas)
├── searches.py         # Operaciones de lectura
├── updates.py          # Operaciones de escritura
├── helpers.py          # Helpers de comparación
└── utils.py            # Utilidades internas
```

### Flujo de operaciones

```
┌─────────────────┐
│  Usuario/Lambda │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ddb_client API │
└────────┬────────┘
         │
         ├─► config.py (validación)
         ├─► utils.py (schema, conversión)
         └─► boto3 (DynamoDB)
```

## ⚙️ Configuración

### config.py

Archivo de configuración central para el módulo:

```python
# config.py

# ═══════════════════════════════════════════════════════════
# CONFIGURACIÓN DE PAGINACIÓN
# ═══════════════════════════════════════════════════════════

# Habilitar/deshabilitar paginación global
paginate = True

# Número de registros por página
pages = 50

# ═══════════════════════════════════════════════════════════
# CAMPOS POR DEFECTO PARA BÚSQUEDAS
# ═══════════════════════════════════════════════════════════

fields = {
    # Tabla de certificaciones
    'bas_certifications': [
        'id',
        'user_id',
        'curso_id',
        'estado',
        'score',
        'created_at',
        'updated_at'
    ],
    
    # Tabla de usuarios
    'bas_users': [
        'user_id',
        'user_name',
        'email',
        'created_at',
        'last_login'
    ],
    
    # Tabla sin campos específicos (retorna todos)
    'bas_portfolio': []
}

# ═══════════════════════════════════════════════════════════
# CAMPOS PERMITIDOS PARA ACTUALIZACIONES
# ═══════════════════════════════════════════════════════════

allowed_fields = {
    'bas_certifications': [
        'estado',
        'score',
        'completed_at',
        'progress',
        'notes',
        'metadata',
        'metadata.course_data',
        'metadata.user_data'  # Soporte para campos anidados
    ],
    
    'bas_users': [
        'user_name',
        'email',
        'cellphone',
        'image',
        'last_login',
        'preferences',
        'preferences.language',
        'preferences.notifications'
    ]
}

# ═══════════════════════════════════════════════════════════
# VALORES POR DEFECTO AL CREAR REGISTROS
# ═══════════════════════════════════════════════════════════

defaults = {
    'data': {
        'bas_certifications': {
            'estado': 0,           # Estado inicial: inactivo
            'score': 0,            # Score inicial: 0
            'progress': 0,         # Progreso inicial: 0%
            'attempts': 0          # Intentos: 0
        },
        
        'bas_users': {
            'role': 'student',
            'active': 1,
            'verified': 0
        },
        
        # Tabla sin defaults
        'bas_portfolio': {}
    }
}
```

### Palabras Reservadas

El módulo maneja automáticamente las palabras reservadas de DynamoDB. Si usas una palabra reservada como nombre de campo, se maneja internamente:

```python
# ❌ boto3 directo - ERROR
table.query(KeyConditionExpression=Key('user_id').eq('123') & Key('status').eq(1))

# ✅ ddb_client - Funciona automáticamente
dynamo_search('table', {'user_id': '123', 'status': 1})
```

La lista completa de palabras reservadas está en `constants.py` (108 palabras).

## 🔍 Módulo Searches

Operaciones de lectura y consulta en DynamoDB.

### Imports

```python
from ddb_client.searches import (
    dynamo_search,      # Búsqueda principal con filtros
    dynamo_counter,     # Contador de registros
    batch_get_items     # Obtener múltiples items
)
```

---

### dynamo_search

Función principal para realizar búsquedas en DynamoDB.

#### Firma

```python
def dynamo_search(
    table: str,
    params: dict,
    limit: bool = False,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla DynamoDB | ✅ |
| `params` | dict | Parámetros de búsqueda (keys + filtros) | ✅ |
| `limit` | bool | Habilitar paginación | ❌ |
| `index` | str | Nombre del índice GSI/LSI | ❌ |
| `order` | str | Orden: 'ASC' o 'DESC' | ❌ |
| `order_by` | str | LSI para ordenar | ❌ |
| `pages` | int | Registros por página | ❌ |
| `log_errors` | bool | Mostrar errores (default: True) | ❌ |
| `log_qparams` | bool | Mostrar query construido | ❌ |
| `log_schema` | bool | Mostrar esquema de tabla | ❌ |

#### Retorno

```python
{
    "Items": [...],                # Lista de items encontrados
    "Count_Items": 10,             # Cantidad de items
    "LastEvaluatedKey": {...}      # Clave para siguiente página (opcional)
}
```

#### Ejemplos

**Búsqueda simple por PK**

```python
params = {
    'user_id': '425034'
}

result = dynamo_search('bas_certifications', params)
# Retorna todos los registros del usuario
```

**Búsqueda por PK y SK**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234'
}

result = dynamo_search('bas_certifications', params)
# Retorna el registro específico
```

**Búsqueda con campos específicos**

```python
params = {
    'user_id': '425034',
    'fields': ['id', 'curso_id', 'estado', 'score']
}

result = dynamo_search('bas_certifications', params, limit=True)
# Solo retorna los campos especificados
```

**Búsqueda con filtros**

```python
from ddb_client.helpers import DynamoComparison as dc

params = {
    'user_id': '425034',
    'estado': dc.Ne(-1),              # Estado diferente de -1
    'score': dc.Ge(80),               # Score >= 80
    'created_at': dc.BeginsWith('2024')  # Creados en 2024
}

result = dynamo_search('bas_certifications', params)
```

**Búsqueda con índice GSI**

```python
params = {
    'curso_id': '1234',     # PK del índice
    'estado': 1             # FK del índice
}

result = dynamo_search(
    'bas_certifications',
    params,
    index='curso_estado-index'
)
```

**Búsqueda con ordenamiento**

```python
params = {
    'user_id': '425034'
}

# Orden descendente
result = dynamo_search(
    'bas_certifications',
    params,
    order='DESC',
    limit=True,
    pages=10
)

# Ordenar por LSI
result = dynamo_search(
    'bas_certifications',
    params,
    order_by='user_date-index'  # LSI name
)
```

**Paginación**

```python
# Primera página
params = {'user_id': '425034'}
result = dynamo_search('bas_certifications', params, limit=True, pages=20)

# Siguiente página
if 'LastEvaluatedKey' in result:
    params['LastEvaluatedKey'] = result['LastEvaluatedKey']
    next_page = dynamo_search('bas_certifications', params, limit=True, pages=20)
```

**Debug con logs**

```python
result = dynamo_search(
    'bas_certifications',
    {'user_id': '425034', 'estado': dc.Ge(0)},
    log_qparams=True,   # Ver query construido
    log_schema=True,    # Ver esquema de tabla
    log_errors=True     # Ver errores detallados
)
```

---

### dynamo_counter

Cuenta el número total de registros que cumplen una condición.

#### Firma

```python
def dynamo_counter(
    table: str,
    params: dict,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `params` | dict | Parámetros de búsqueda | ✅ |
| `log_qparams` | bool | Mostrar query | ❌ |
| `log_schema` | bool | Mostrar esquema | ❌ |
| `log_errors` | bool | Mostrar errores | ❌ |

#### Retorno

```python
{
    "Count_Items": 150,    # Total de items
    "Pages": 3             # Número de páginas (según config.pages)
}
```

#### Ejemplos

```python
# Contar todos los registros de un usuario
params = {'user_id': '425034'}
result = dynamo_counter('bas_certifications', params)
print(f"Total: {result['Count_Items']}, Páginas: {result['Pages']}")

# Contar con filtros
from ddb_client.helpers import DynamoComparison as dc

params = {
    'user_id': '425034',
    'estado': dc.Eq(1),  # Solo activos
    'score': dc.Ge(70)   # Score >= 70
}

result = dynamo_counter('bas_certifications', params)
```

---

### batch_get_items

Obtiene múltiples items en una sola operación usando sus claves primarias.

#### Firma

```python
def batch_get_items(
    table: str,
    source: list,
    **args
) -> list
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `source` | list | Lista de items con las claves | ✅ |
| `{pk_name}` | str | Mapeo de PK (key=valor) | ✅ |
| `{sk_name}` | str | Mapeo de SK (key=valor) | ❌ |
| `fields` | list | Campos a retornar | ❌ |
| `log_keys` | bool | Log de claves procesadas | ❌ |
| `log_result` | bool | Log del resultado | ❌ |

#### Retorno

```python
[
    {...},  # Item 1
    {...},  # Item 2
    {...}   # Item N
]
```

#### Ejemplos

**Solo Primary Key**

```python
# Lista de IDs a buscar
source = [
    {'id': 'abc123'},
    {'id': 'def456'},
    {'id': 'ghi789'}
]

# Especificar mapeo: tabla usa 'user_id', source usa 'id'
items = batch_get_items('bas_users', source, user_id='id')
```

**Primary Key + Sort Key**

```python
# Lista con PK y SK
attempts = [
    {'user': '425034', 'course': '4350'},
    {'user': '425034', 'course': '4467'},
    {'user': '568745', 'course': '4350'}
]

# Especificar ambos mapeos
items = batch_get_items(
    'bas_certifications',
    attempts,
    user_id='user',      # PK mapping
    curso_id='course'    # SK mapping
)
```

**Con campos específicos**

```python
attempts = [
    {'user': '425034', 'course': '4350'},
    {'user': '425034', 'course': '4467'}
]

items = batch_get_items(
    'bas_certifications',
    attempts,
    user_id='user',
    curso_id='course',
    fields=['user_id', 'curso_id', 'estado', 'score']
)
```

**Con valores anidados**

```python
# Source con datos anidados
source = [
    {
        'metadata': {
            'user_data': {
                'id': '425034'
            }
        },
        'course_info': {
            'id': '1234'
        }
    }
]

# Usar dot notation para acceder
items = batch_get_items(
    'bas_certifications',
    source,
    user_id='metadata.user_data.id',
    curso_id='course_info.id'
)
```

**Con logging para debug**

```python
items = batch_get_items(
    'bas_certifications',
    attempts,
    user_id='user_id',
    curso_id='curso_id',
    log_keys=True,      # Ver claves procesadas
    log_result=True     # Ver respuesta DynamoDB
)
```

#### Límites

- Máximo **100 items** por batch (límite de DynamoDB)
- Automáticamente divide en múltiples batches si es necesario
- Maneja `UnprocessedKeys` automáticamente con reintentos

---

## 📝 Módulo Updates

Operaciones de escritura en DynamoDB.

### Imports

```python
from ddb_client.updates import (
    dynamo_create,       # Crear registro
    batch_create_items,  # Crear múltiples registros
    dynamo_update,       # Actualizar registro
    dynamo_increase,     # Incrementar campos
    dynamo_delete,       # Eliminar registro
    batch_delete_items   # Eliminar múltiples registros
)
```

---

### dynamo_create

Crea un nuevo registro en DynamoDB.

#### Firma

```python
def dynamo_create(
    table: str,
    data: dict,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `data` | dict | Datos del registro | ✅ |
| `log_qparams` | bool | Log del item | ❌ |
| `log_errors` | bool | Log de errores | ❌ |

#### Retorno

```python
{
    "data": {...},        # Item creado completo
    "status": True        # True si fue exitoso
}
```

#### Campos Automáticos

La función agrega automáticamente (si no existen en `data`):

```python
{
    'id': 'uuid4-generado',
    'timestamp': 1234567890123,  # Unix timestamp en ms
    'created_at': '2024-01-15T10:30:00.000Z'  # ISO 8601 UTC
}
```

#### Ejemplos

**Creación básica**

```python
data = {
    'user_id': '425034',
    'curso_id': '1234',
    'name': 'Curso de Python'
}

result = dynamo_create('bas_certifications', data)

if result['status']:
    print(f"Creado con ID: {result['data']['id']}")
```

**Con valores por defecto**

Los valores en `config.defaults` se aplican automáticamente:

```python
# config.py tiene:
# defaults['data']['bas_certifications'] = {'estado': 0, 'score': 0}

data = {
    'user_id': '425034',
    'curso_id': '1234'
}

result = dynamo_create('bas_certifications', data)
# result['data'] incluirá: estado=0, score=0
```

**Sobrescribir campos automáticos**

```python
data = {
    'id': 'custom-id-123',           # Usar ID personalizado
    'user_id': '425034',
    'curso_id': '1234',
    'created_at': '2024-01-01T00:00:00Z'  # Fecha personalizada
}

result = dynamo_create('bas_certifications', data)
# Respeta los valores proporcionados
```

---

### batch_create_items

Crea múltiples registros en una sola operación.

#### Firma

```python
def batch_create_items(
    table: str,
    source: list,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `source` | list | Lista de items a crear | ✅ |
| `{pk_name}` | str | Mapeo de PK | ✅ |
| `{sk_name}` | str | Mapeo de SK | ❌ |
| `log_keys` | bool | Log de items | ❌ |
| `log_result` | bool | Log de respuesta | ❌ |

#### Retorno

```python
{
    "status": True,      # True si alguno fue exitoso
    "row_count": 25      # Número de registros creados
}
```

#### Ejemplos

```python
items = [
    {'user': '123', 'course': '456', 'name': 'Item 1'},
    {'user': '123', 'course': '789', 'name': 'Item 2'},
    {'user': '456', 'course': '456', 'name': 'Item 3'}
]

result = batch_create_items(
    'bas_certifications',
    items,
    user_id='user',
    curso_id='course'
)

print(f"Creados: {result['row_count']} registros")
```

#### Límites

- Máximo **25 items** por batch (límite DynamoDB)
- Automáticamente divide en múltiples batches
- Maneja `UnprocessedItems` con reintentos

---

### dynamo_update

Actualiza un registro existente.

#### Firma

```python
def dynamo_update(
    table: str,
    keys: dict,
    data: dict,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `keys` | dict | Claves del registro (PK + SK) | ✅ |
| `data` | dict | Campos a actualizar | ✅ |
| `{field}` | any | Condiciones (kwargs) | ❌ |
| `log_qparams` | bool | Log de query | ❌ |
| `log_schema` | bool | Log de esquema | ❌ |
| `log_keys` | bool | Log de claves | ❌ |
| `log_criticals` | bool | Log de campos filtrados | ❌ |

#### Retorno

```python
{
    "status": True,      # True si fue exitoso
    "row_count": 1       # 1 si se actualizó, 0 si no
}
```

#### Ejemplos

**Actualización simple**

```python
keys = {
    'user_id': '425034',
    'curso_id': '1234'
}

data = {
    'estado': 1,
    'score': 95,
    'completed_at': '2024-01-15T10:00:00Z'
}

result = dynamo_update('bas_certifications', keys, data)
```

**Con actualización condicional**

```python
keys = {
    'user_id': '425034',
    'curso_id': '1234'
}

data = {
    'score': 100
}

# Solo actualizar si estado = 1
result = dynamo_update(
    'bas_certifications',
    keys,
    data,
    estado=1  # Condición
)
```

**Actualizar campos anidados**

```python
# config.py debe incluir el campo anidado en allowed_fields
# allowed_fields['table'] = ['metadata.course_data']

data = {
    'metadata.course_data': {'progress': 75, 'last_lesson': 10}
}

result = dynamo_update('bas_certifications', keys, data)
```

**Protección de campos no permitidos**

```python
# Si un campo NO está en config.allowed_fields, se filtra:

data = {
    'score': 95,              # ✅ Permitido
    'user_id': 'other-user',  # ❌ Filtrado (no permitido)
    'secret_field': 'xxx'     # ❌ Filtrado
}

result = dynamo_update('bas_certifications', keys, data, log_criticals=True)
# Log: "Filtered fields not allowed: ['user_id', 'secret_field']"
```

---

### dynamo_increase

Incrementa o decrementa campos numéricos de forma atómica.

#### Firma

```python
def dynamo_increase(
    table: str,
    params: dict,
    data: dict,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `params` | dict | Claves del registro + límites | ✅ |
| `data` | dict | Campos a incrementar | ✅ |
| `log_qparams` | bool | Log de query | ❌ |
| `log_schema` | bool | Log de esquema | ❌ |
| `log_errors` | bool | Log de errores | ❌ |

#### Retorno

```python
{
    "status": True,      # True si fue exitoso
    "row_count": 1       # 1 si se actualizó
}
```

#### Ejemplos

**Incremento simple**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234'
}

data = {
    'score': 5,      # Incrementar 5 puntos
    'attempts': 1    # Incrementar 1 intento
}

result = dynamo_increase('bas_certifications', params, data)
```

**Decremento**

```python
data = {
    'lives': -1,      # Decrementar 1 vida
    'credits': -10    # Decrementar 10 créditos
}

result = dynamo_increase('bas_game_state', params, data)
```

**Con límite máximo**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234',
    'score_max': 100  # No permitir score > 100
}

data = {
    'score': 10  # Intentar incrementar 10
}

result = dynamo_increase('bas_certifications', params, data)
# Si score actual es 95, solo se incrementará a 100
```

**Con límite mínimo**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234',
    'lives_min': 0  # No permitir vidas negativas
}

data = {
    'lives': -2  # Intentar decrementar 2
}

result = dynamo_increase('bas_game_state', params, data)
# Si lives actual es 1, la operación fallará
```

**Múltiples campos con límites**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234',
    'score_min': 0,
    'score_max': 100,
    'attempts_min': 0,
    'attempts_max': 10
}

data = {
    'score': 5,
    'attempts': 1
}

result = dynamo_increase('bas_certifications', params, data)
```

---

### dynamo_delete

Elimina un registro de DynamoDB.

#### Firma

```python
def dynamo_delete(
    table: str,
    params: dict,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `params` | dict | Claves del registro | ✅ |
| `return_values` | bool | Retornar valores antes de eliminar | ❌ |
| `log_qparams` | bool | Log de query | ❌ |
| `log_schema` | bool | Log de esquema | ❌ |
| `log_errors` | bool | Log de errores | ❌ |

#### Retorno

```python
{
    "status": True,      # True si fue exitoso
    "row_count": 1       # 1 si se eliminó, 0 si no existía
}
```

#### Ejemplos

**Eliminación simple**

```python
params = {
    'user_id': '425034',
    'curso_id': '1234'
}

result = dynamo_delete('bas_certifications', params)

if result['status']:
    print("Registro eliminado")
```

**Sin retornar valores**

```python
result = dynamo_delete(
    'bas_certifications',
    params,
    return_values=False
)
```

---

### batch_delete_items

Elimina múltiples registros en una sola operación.

#### Firma

```python
def batch_delete_items(
    table: str,
    source: list,
    **args
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `table` | str | Nombre de la tabla | ✅ |
| `source` | list | Lista de claves a eliminar | ✅ |
| `{pk_name}` | str | Mapeo de PK | ✅ |
| `{sk_name}` | str | Mapeo de SK | ❌ |
| `log_keys` | bool | Log de claves | ❌ |
| `log_result` | bool | Log de respuesta | ❌ |

#### Retorno

```python
{
    "status": True,      # True si alguno fue eliminado
    "row_count": 15      # Número de registros eliminados
}
```

#### Ejemplos

```python
to_delete = [
    {'user_id': '123', 'curso_id': '456'},
    {'user_id': '123', 'curso_id': '789'},
    {'user_id': '456', 'curso_id': '123'}
]

result = batch_delete_items(
    'bas_certifications',
    to_delete,
    user_id='user_id',
    curso_id='curso_id'
)

print(f"Eliminados: {result['row_count']} registros")
```

---

## 🛠️ Helpers

### DynamoComparison

Clase para construir expresiones de comparación avanzadas.

#### Imports

```python
from ddb_client.helpers import DynamoComparison as dc
```

#### Operadores de Comparación

| Método | Descripción | Ejemplo |
|--------|-------------|---------|
| `Eq(value)` | Igual a | `dc.Eq('active')` |
| `Ne(value)` | Diferente de | `dc.Ne(-1)` |
| `Lt(value)` | Menor que | `dc.Lt(100)` |
| `Le(value)` | Menor o igual | `dc.Le(100)` |
| `Gt(value)` | Mayor que | `dc.Gt(0)` |
| `Ge(value)` | Mayor o igual | `dc.Ge(70)` |
| `Between(start, end)` | Entre dos valores | `dc.Between(10, 100)` |
| `BeginsWith(value)` | Comienza con | `dc.BeginsWith('2024')` |
| `Contains(value)` | Contiene | `dc.Contains('python')` |
| `NotContains(value)` | No contiene | `dc.NotContains('test')` |
| `In(values)` | En lista | `dc.In(['a', 'b', 'c'])` |
| `Null()` | Es nulo | `dc.Null()` |
| `NotNull()` | No es nulo | `dc.NotNull()` |

#### Operadores Lógicos

**OR**

```python
params = {
    'user_id': '425034',
    'estado': dc.Or([
        dc.Eq('active'),
        dc.Eq('pending'),
        dc.Eq('processing')
    ])
}
```

**AND**

```python
params = {
    'user_id': '425034',
    'score': dc.And([
        dc.Ge(70),     # score >= 70
        dc.Le(100)     # score <= 100
    ])
}
```

**Combinaciones complejas**

```python
params = {
    'user_id': '425034',
    'estado': dc.Or([
        dc.Eq('active'),
        dc.Eq('pending')
    ]),
    'score': dc.And([
        dc.Ge(70),
        dc.Lt(100)
    ]),
    'created_at': dc.BeginsWith('2024')
}
```

#### Uso con Sort Keys

```python
# Sort Key con comparador
params = {
    'user_id': '425034',
    'curso_id': dc.BeginsWith('course_', is_key=True)
}

result = dynamo_search('bas_certifications', params)
```

---

## 🐛 Solución de Problemas

### Error: "Key not found in schema"

**Causa**: La clave especificada no existe en el esquema de la tabla.

**Solución**:

```python
# ❌ Incorrecto
params = {'user_id': '123'}
dynamo_search('table_con_otro_pk', params)

# ✅ Correcto - verificar esquema primero
result = dynamo_search('table', params, log_schema=True)
# Revisar los logs para ver el esquema real
```

### Error: "Field not allowed"

**Causa**: Intentas actualizar un campo no incluido en `config.allowed_fields`.

**Solución**:

```python
# Agregar el campo a config.py
allowed_fields = {
    'tu_tabla': [
        'campo_existente',
        'campo_nuevo'  # ← Agregar aquí
    ]
}
```

### Error: "ConditionalCheckFailedException"

**Causa**: En `dynamo_increase`, se alcanzó un límite min/max.

**Solución**:

```python
# Verificar límites
params = {
    'user_id': '123',
    'score_max': 100
}
data = {'score': 10}

try:
    result = dynamo_increase('table', params, data)
except Exception as e:
    if 'ConditionalCheckFailedException' in str(e):
        print("Límite alcanzado")
```

### Paginación infinita

**Causa**: No se maneja correctamente `LastEvaluatedKey`.

**Solución**:

```python
# ✅ Siempre verificar antes del siguiente loop
while True:
    result = dynamo_search(table, params, limit=True)
    
    # Procesar items...
    
    # ✅ Verificar si hay más páginas
    if 'LastEvaluatedKey' not in result:
        break  # ← IMPORTANTE
    
    params['LastEvaluatedKey'] = result['LastEvaluatedKey']
```

### Performance lento en búsquedas

**Causas y soluciones**:

1. **Filtros en campos no indexados**
   ```python
   # ❌ Lento - filtra después de obtener todos
   params = {'user_id': '123', 'some_field': 'value'}
   
   # ✅ Rápido - usar índice GSI
   params = {'indexed_field': 'value'}
   dynamo_search(table, params, index='gsi_name')
   ```

2. **Retornando demasiados campos**
   ```python
   # ❌ Lento - retorna todo
   result = dynamo_search(table, params)
   
   # ✅ Rápido - solo campos necesarios
   params = {'user_id': '123', 'fields': ['id', 'name']}
   result = dynamo_search(table, params)
   ```

3. **Sin límite de paginación**
   ```python
   # ❌ Lento - obtiene todo de una vez
   result = dynamo_search(table, params, limit=False)
   
   # ✅ Rápido - paginar resultados
   result = dynamo_search(table, params, limit=True, pages=50)
   ```

---

## 📚 Referencias

- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [Boto3 DynamoDB Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)

---

**[⬅️ Volver al README principal](../README.md)**


# MySQL Client Layer (Python 3.12 / x86_64 + arm64)

Cliente MySQL pensado para usarse como AWS Lambda Layer. Incluye helpers CRUD y utilidades seguras y simples para consultar, insertar, actualizar, eliminar e incrementar campos.

- Runtime: Python 3.12
- Arquitecturas: x86_64 y arm64

---

## Tabla de Contenidos

- [Variables de Entorno](#variables-de-entorno)
- [Archivo config.py](#archivo-configpy)
- [Importación](#importación)
- [Búsquedas (searches.py)](#búsquedas-searchespy)
  - [search](#search)
  - [counter](#counter)
- [Actualizaciones (updates.py)](#actualizaciones-updatespy)
  - [create](#create)
  - [update](#update)
  - [delete](#delete)
  - [increase](#increase)

---

## Variables de Entorno

Configura en la Lambda:

- `DB_USER`: Usuario
- `DB_PASSWORD`: Contraseña
- `DB_NAME`: Nombre de la base de datos
- `DB_ENDPOINT_RO` (opcional): Endpoint de solo lectura
- `DB_ENDPOINT_RW` (opcional): Endpoint de lectura/escritura

---

## Archivo config.py

Crea un archivo `config.py` en la raíz de tu Lambda con los diccionarios de control:

```python
# Campos a devolver por tabla en SELECT (vacío o no definido = *)
rds_fields = {
    'users': ['id', 'name', 'email', 'status'],
    'orders': [],  # array vacío => traer todos los campos
    # 'products' no definido => traer todos los campos
}

# Campos que se pueden ACTUALIZAR/INCREMENTAR por tabla
rds_allowed_fields = {
    'users': ['name', 'email', 'status', 'attempts', 'score', 'updated_at'],
    'orders': ['status', 'total', 'updated_at'],
}

# Valores por defecto para INSERT
rds_defaults = {
    'data': {
        'users': {'status': 'active', 'role': 'user'},
        'orders': {'status': 'pending', 'currency': 'USD'}
    }
}
```

---

## Importación

```python
from mysql_client.searches import search, counter
from mysql_client.updates import create, update, delete, increase
```

---

## Búsquedas (searches.py)

### search

Realiza un SELECT con soporte de operadores y control de columnas vía `rds_fields`.

Operadores soportados en `params`:
- Igualdad (por defecto): `{ "status": "active" }`
- Comparaciones: `{ "age": {">": 18} }`, `{ "price": {"<=": 100} }`
- Distinto: `{ "status": {"!=": "deleted"} }`
- LIKE/NOT LIKE: `{ "name": {"LIKE": "%John%"} }`
- BETWEEN: `{ "age": {"BETWEEN": [18, 65]} }`
- IN / NOT IN: `{ "id": {"IN": [1,2,3]} }`

Ejemplos:

```python
# Búsqueda simple
users = search('users', { 'status': 'active', 'country': 'US' })

# Búsqueda avanzada
users = search('users', {
    'age': {'>': 18},
    'name': {'LIKE': '%John%'}
})

# Rango y conjuntos
products = search('products', {
    'price': {'BETWEEN': [10, 100]},
    'category': {'IN': ['electronics', 'books']}
})
```

Notas:
- Si la tabla no está en `rds_fields` o su lista está vacía → `SELECT *`.
- Usa endpoint RO por defecto.

### counter

Devuelve solo el número de filas que cumplen la condición (no usa `rds_fields`).

```python
# Conteo simple
total = counter('users', { 'status': 'active' })

# Conteo con operadores
pending = counter('orders', {
    'created_at': {'BETWEEN': ['2025-01-01', '2025-12-31']},
    'status': {'NOT IN': ['cancelled', 'refunded']}
})
```

---

## Actualizaciones (updates.py)

### create

Inserta un registro aplicando valores por defecto desde `rds_defaults['data'][table]`.

```python
new_id = create('users', {
    'name': 'John',
    'email': 'john@example.com'
})
# Si hay defaults (p.ej. status='active'), se aplican automáticamente
```

Requiere endpoint RW.

### update

Actualiza campos permitidos según `rds_allowed_fields[table]`. Rechaza `params` vacío (para evitar updates masivos).

```python
affected = update('users', { 'id': 123 }, { 'status': 'verified', 'email': 'new@example.com' })
```

Requiere endpoint RW.

### delete

Elimina filas filtradas por `params`. Rechaza `params` vacío (para evitar deletes masivos).

```python
deleted = delete('sessions', { 'status': 'expired', 'user_id': 42 })
```

Requiere endpoint RW.

### increase

Incrementa/decrementa múltiples campos de forma atómica. Solo opera sobre campos permitidos en `rds_allowed_fields[table]`. Soporta límites mínimos y máximos definidos en `params` usando sufijos `_<min|max>`.

- `score_max`: No permitir que `score` supere ese valor
- `lives_min`: No permitir que `lives` baje de ese valor

```python
# Incremento simple
increase('bas_certifications',
    { 'user_id': '425034', 'curso_id': '1234' },
    { 'score': 5, 'attempts': 1 }
)

# Decremento
increase('bas_game_state',
    { 'user_id': '425034' },
    { 'lives': -1, 'credits': -10 }
)

# Con límite máximo
increase('bas_certifications',
    { 'user_id': '425034', 'score_max': 100 },
    { 'score': 10 }
)
# Si score actual es 95, quedará en 100

# Con límite mínimo
increase('bas_game_state',
    { 'user_id': '425034', 'lives_min': 0 },
    { 'lives': -2 }
)
# Si lives es 1, quedará en 0

# Múltiples campos con límites
increase('bas_certifications',
    {
      'user_id': '425034',
      'score_min': 0, 'score_max': 100,
      'attempts_min': 0, 'attempts_max': 10
    },
    { 'score': 5, 'attempts': 1 }
)
```

Requiere endpoint RW.

---

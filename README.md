# AWS Edutin Layer

[![Python Version](https://img.shields.io/badge/python-3.10%20|%203.11%20|%203.12-blue.svg)](https://www.python.org/downloads/)
[![AWS Lambda](https://img.shields.io/badge/AWS-Lambda%20Layer-orange.svg)](https://aws.amazon.com/lambda/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

> **Capa de AWS Lambda profesional para simplificar las interacciones con DynamoDB, OpenSearch y otros servicios de AWS en el ecosistema Edutin.**

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Características](#características)
- [Arquitectura](#arquitectura)
- [Instalación](#instalación)
- [Inicio Rápido](#inicio-rápido)
- [Módulos](#módulos)
  - [ddb_client](#ddb_client)
  - [aws_client](#aws_client)
  - [utils](#utils)
- [Configuración](#configuración)
- [Deployment](#deployment)
- [Ejemplos de Uso](#ejemplos-de-uso)
- [Mejores Prácticas](#mejores-prácticas)
- [Contribución](#contribución)
- [Licencia](#licencia)

## 🎯 Descripción

**AWS Edutin Layer** es una capa de Lambda altamente optimizada diseñada para simplificar y estandarizar las operaciones con servicios AWS en proyectos de Edutin. Esta capa proporciona abstracciones de alto nivel que reducen la complejidad del código y mejoran la mantenibilidad de las funciones Lambda.

### ¿Por qué usar esta capa?

- ✅ **Abstracción simplificada**: Operaciones complejas de DynamoDB y OpenSearch reducidas a simples llamadas de función
- ✅ **Validación integrada**: Validación automática de esquemas y tipos de datos
- ✅ **Logging estructurado**: Sistema de logs consistente y trazable
- ✅ **Optimización de rendimiento**: Operaciones batch y manejo eficiente de recursos
- ✅ **Manejo de errores robusto**: Gestión centralizada de excepciones y reintentos
- ✅ **Type hints completos**: Documentación y autocompletado en IDEs modernos

## ✨ Características

### DynamoDB Client (`ddb_client`)

- **Búsquedas avanzadas**: Queries con filtros, ordenamiento y paginación
- **Operaciones batch**: Inserción y eliminación masiva optimizada
- **Actualizaciones incrementales**: Contadores atómicos con límites min/max
- **Helpers de comparación**: Operadores lógicos (OR, AND) y comparadores dinámicos
- **Validación de esquemas**: Verificación automática contra esquemas definidos

### AWS Client (`aws_client`)

- **Lambda Functions**: Invocación de funciones, gestión de metadata
- **EventBridge**: Envío de eventos estructurados
- **WebSocket API**: Comunicación en tiempo real con clientes
- **OpenSearch**: CRUD completo, búsquedas full-text, índices optimizados
- **Normalización de texto**: Procesamiento de texto con soporte para español

### Utilidades (`utils`)

- **Logger runtime**: Captura automática de contexto de ejecución
- **Structured logging**: Logs JSON con contexto completo

## 🏗️ Arquitectura

```
aws_edutin_layer/
├── python/
│   └── python/
│       ├── ddb_client/          # Cliente DynamoDB
│       │   ├── __init__.py
│       │   ├── config.py        # Configuración de tablas
│       │   ├── constants.py     # Palabras reservadas DynamoDB
│       │   ├── searches.py      # Operaciones de búsqueda
│       │   ├── updates.py       # Operaciones de escritura
│       │   ├── helpers.py       # Helpers de comparación
│       │   └── utils.py         # Utilidades internas
│       │
│       ├── aws_client/          # Cliente AWS Services
│       │   ├── aws_lambda.py    # Lambda & EventBridge
│       │   └── open_search/     # OpenSearch operations
│       │       ├── __init__.py
│       │       ├── open_search_module.py  # Cliente base
│       │       ├── op_searches.py         # Búsquedas
│       │       ├── op_updates.py          # CRUD operations
│       │       └── op_querys.py           # Query builders
│       │
│       └── utils/               # Utilidades generales
│           └── logs.py          # Sistema de logging
│
├── deploy.py                    # Script de deployment
├── messages.py                  # Mensajes de deployment
└── README.md                    # Este archivo
```

## 📦 Instalación

### Como Lambda Layer

Esta capa está diseñada para ser desplegada como una AWS Lambda Layer:

```bash
# Clonar el repositorio
git clone <repository-url>
cd aws_edutin_layer

# Desplegar a desarrollo
python deploy.py --env develop

# Desplegar a producción (requiere rama master)
python deploy.py --env production --message "Descripción del cambio"
```

### Para desarrollo local

```bash
# Instalar dependencias
pip install -r requirements.txt

# O usar el código directamente
export PYTHONPATH="${PYTHONPATH}:/ruta/al/proyecto/python"
```

## 🚀 Inicio Rápido

### Ejemplo 1: Búsqueda en DynamoDB

```python
from ddb_client.searches import dynamo_search
from ddb_client.helpers import DynamoComparison as dc

# Búsqueda simple
params = {
    'user_id': '425034',
    'curso_id': '1234'
}
result = dynamo_search('bas_certifications', params)
print(f"Encontrados: {result['Count_Items']} items")

# Búsqueda con filtros avanzados
params = {
    'user_id': '425034',
    'estado': dc.Ne(-1),  # Estado diferente de -1
    'score': dc.Between(100, 500)  # Score entre 100 y 500
}
result = dynamo_search('bas_certifications', params, limit=True)
```

### Ejemplo 2: Crear registro en DynamoDB

```python
from ddb_client.updates import dynamo_create

data = {
    'user_id': '425034',
    'curso_id': '5678',
    'name': 'Curso de Python',
    'estado': 1,
    'score': 95
}

result = dynamo_create('bas_certifications', data)
if result['status']:
    print(f"Registro creado: {result['data']['id']}")
```

### Ejemplo 3: Búsqueda en OpenSearch

```python
from aws_client.open_search.op_searches import op_search

# Buscar cursos
result = op_search(
    index_name='courses',
    query='python programming',
    size=20
)

print(f"Total encontrados: {result['total_hits']}")
for course in result['courses']:
    print(f"- {course['name']} ({course['students']} estudiantes)")
```

### Ejemplo 4: Invocar otra Lambda

```python
from aws_client.aws_lambda import invoke_lambda, get_lambda_metadata

def lambda_handler(event, context):
    # Obtener metadata de la función actual
    metadata = get_lambda_metadata(context)
    
    # Invocar otra función
    payload = {
        'action': 'process_data',
        'data': {'user_id': '123'}
    }
    
    result = invoke_lambda(
        lambda_name='processor-function',
        payload=payload,
        invocation_type='RequestResponse',
        qualifier=metadata['Alias']
    )
    
    return result
```

## 📚 Módulos

### ddb_client

Cliente completo para operaciones con DynamoDB.

#### Funciones principales

##### Búsquedas

```python
from ddb_client.searches import (
    dynamo_search,      # Búsqueda con filtros y paginación
    dynamo_counter,     # Contador de registros
    batch_get_items     # Obtener múltiples items por clave
)
```

##### Actualizaciones

```python
from ddb_client.updates import (
    dynamo_create,       # Crear registro
    batch_create_items,  # Crear múltiples registros
    dynamo_update,       # Actualizar registro
    dynamo_increase,     # Incrementar campos numéricos
    dynamo_delete,       # Eliminar registro
    batch_delete_items   # Eliminar múltiples registros
)
```

##### Helpers

```python
from ddb_client.helpers import DynamoComparison as dc

# Operadores de comparación
dc.Eq(value)           # Igual a
dc.Ne(value)           # Diferente de
dc.Lt(value)           # Menor que
dc.Le(value)           # Menor o igual
dc.Gt(value)           # Mayor que
dc.Ge(value)           # Mayor o igual
dc.Between(start, end) # Entre dos valores
dc.BeginsWith(value)   # Comienza con
dc.Contains(value)     # Contiene
dc.In(values)          # En lista
dc.Null()              # Es nulo
dc.NotNull()           # No es nulo

# Operadores lógicos
dc.Or([condition1, condition2])   # OR lógico
dc.And([condition1, condition2])  # AND lógico
```

**[📖 Ver documentación completa de ddb_client](docs/DDB_CLIENT.md)**

### aws_client

Cliente para servicios AWS (Lambda, EventBridge, OpenSearch).

#### Lambda & EventBridge

```python
from aws_client.aws_lambda import (
    get_lambda_metadata,    # Obtener metadata de Lambda
    invoke_lambda,          # Invocar otra Lambda
    invoke_event_bus,       # Enviar evento a EventBridge
    send_webSocket_message  # Enviar mensaje WebSocket
)
```

#### OpenSearch

```python
from aws_client.open_search.op_searches import op_search
from aws_client.open_search.op_updates import (
    op_create,       # Crear documento
    op_update,       # Actualizar documento
    op_delete,       # Eliminar documento
    op_bulk_create   # Crear múltiples documentos
)
```

**[📖 Ver documentación completa de aws_client](docs/AWS_CLIENT.md)**

### utils

Utilidades generales para Lambda functions.

```python
from utils.logs import logger_runtime_event

def lambda_handler(event, context):
    # Log automático del evento con contexto
    logger_runtime_event(event)
    
    # Tu código aquí...
```

**[📖 Ver documentación completa de utils](docs/UTILS.md)**

## ⚙️ Configuración

### config.py

Configuración global para `ddb_client`:

```python
# config.py

# Paginación
paginate = True
pages = 50  # Registros por página

# Campos por defecto a retornar en búsquedas
fields = {
    'bas_certifications': [
        'user_id',
        'curso_id',
        'estado',
        'created_at'
    ],
    'bas_users': []  # Vacío = todos los campos
}

# Campos permitidos para actualizaciones
allowed_fields = {
    'bas_certifications': [
        'estado',
        'score',
        'completed_at'
    ]
}

# Valores por defecto al crear registros
defaults = {
    'data': {
        'bas_certifications': {
            'estado': 0,
            'score': 0
        },
        'bas_users': {}
    }
}
```

### Variables de entorno

Para OpenSearch, configura estas variables:

```bash
OPENSEARCH_HOST=your-opensearch-endpoint
OPENSEARCH_PORT=9200
OPENSEARCH_USERNAME=admin
OPENSEARCH_PASSWORD=your-password
OPENSEARCH_KEY_ACCESS=your-api-key
```

## 🚢 Deployment

### Requisitos previos

- AWS CLI configurado con credenciales
- Python 3.10, 3.11 o 3.12
- Git (para control de versiones)

### Comandos de deployment

```bash
# Desarrollo (desde cualquier rama)
python deploy.py --env develop

# Producción (solo desde rama master)
git checkout master
python deploy.py --env production --message "Descripción del cambio"

# Deployment con logs verbose
python deploy.py --env develop --verbose
```

### Estructura del deployment

El script `deploy.py`:
1. ✅ Valida las credenciales AWS
2. 📦 Comprime el directorio `python/`
3. 🚀 Publica nueva versión de la Layer
4. ✨ Retorna la versión publicada

### Configuración en Lambda

Después del deployment, agrega la Layer a tus funciones:

```bash
# AWS CLI
aws lambda update-function-configuration \
  --function-name tu-funcion \
  --layers arn:aws:lambda:region:account:layer:EdutinLayers:VERSION

# Terraform
resource "aws_lambda_function" "example" {
  layers = ["arn:aws:lambda:region:account:layer:EdutinLayers:VERSION"]
}
```

## 💡 Ejemplos de Uso

### Búsqueda avanzada con paginación

```python
from ddb_client.searches import dynamo_search
from ddb_client.helpers import DynamoComparison as dc

params = {
    'user_id': '425034',
    'estado': dc.Or([
        dc.Eq('active'),
        dc.Eq('pending')
    ]),
    'created_at': dc.BeginsWith('2024'),
    'fields': ['id', 'curso_id', 'estado', 'score']
}

# Primera página
result = dynamo_search('bas_certifications', params, limit=True, pages=20)

# Página siguiente
if 'LastEvaluatedKey' in result:
    params['LastEvaluatedKey'] = result['LastEvaluatedKey']
    next_page = dynamo_search('bas_certifications', params, limit=True)
```

### Operaciones batch

```python
from ddb_client.updates import batch_create_items, batch_delete_items

# Crear múltiples registros
items = [
    {'user_id': '123', 'curso_id': '456', 'name': 'Item 1'},
    {'user_id': '123', 'curso_id': '789', 'name': 'Item 2'},
    {'user_id': '123', 'curso_id': '012', 'name': 'Item 3'}
]

result = batch_create_items('bas_certifications', items, user_id='user_id', curso_id='curso_id')
print(f"Creados: {result['row_count']} registros")

# Eliminar múltiples registros
to_delete = [
    {'user_id': '123', 'curso_id': '456'},
    {'user_id': '123', 'curso_id': '789'}
]

result = batch_delete_items('bas_certifications', to_delete, user_id='user_id', curso_id='curso_id')
print(f"Eliminados: {result['row_count']} registros")
```

### Incrementos con límites

```python
from ddb_client.updates import dynamo_increase

# Incrementar score con límite máximo
params = {
    'user_id': '425034',
    'curso_id': '1234',
    'score_max': 100  # No permitir más de 100
}

data = {
    'score': 5  # Incrementar 5 puntos
}

result = dynamo_increase('bas_certifications', params, data)

# Decrementar con límite mínimo
params = {
    'user_id': '425034',
    'curso_id': '1234',
    'attempts_min': 0  # No permitir valores negativos
}

data = {
    'attempts': -1  # Decrementar 1
}

result = dynamo_increase('bas_certifications', params, data)
```

### Búsqueda y actualización en OpenSearch

```python
from aws_client.open_search.op_searches import op_search
from aws_client.open_search.op_updates import op_update

# Buscar cursos
courses = op_search(
    index_name='courses',
    query='machine learning',
    size=10,
    name_boost=30,
    description_boost=5
)

# Actualizar un curso
update_result = op_update(
    index_name='courses',
    document={
        'course_id': '1234',
        'students': 5000,
        'rating': 4.8,
        'description': 'Curso actualizado de ML'
    }
)

if update_result['status']:
    print("Curso actualizado exitosamente")
```

### EventBridge con trazabilidad

```python
from aws_client.aws_lambda import invoke_event_bus, get_lambda_metadata

def lambda_handler(event, context):
    # Obtener metadata para trazabilidad
    lambda_meta = get_lambda_metadata(context)
    
    # Preparar payload
    payload = {
        'user_id': '12345',
        'action': 'purchase',
        'course_id': '6789',
        'amount': 99.99,
        'params': event.get('queryStringParameters', {}),
        'authorizer': event.get('requestContext', {}).get('authorizer', {})
    }
    
    # Enviar evento
    result = invoke_event_bus(
        EventBusName='edutin-events',
        Source='course.purchases',
        payload=payload,
        DetailType='CoursePurchase',
        lambda_details=lambda_meta,
        log_enabled=True
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Event sent',
            'event_id': result.get('FailedEntryCount', 0) == 0
        })
    }
```

## 🤝 Contribución

### Flujo de trabajo

1. **Fork el repositorio**
2. **Crea una rama feature**: `git checkout -b feature/nueva-funcionalidad`
3. **Implementa tus cambios** con tests
4. **Commit con mensaje descriptivo**: `git commit -m 'feat: agregar nueva funcionalidad'`
5. **Push a tu fork**: `git push origin feature/nueva-funcionalidad`
6. **Crea un Pull Request** a `develop`

### Convenciones de código

- **PEP 8**: Seguir las guías de estilo de Python
- **Type Hints**: Usar anotaciones de tipo en todas las funciones públicas
- **Docstrings**: Documentar todas las funciones con formato Google Style
- **Tests**: Incluir tests unitarios para nuevas funcionalidades

### Convenciones de commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` Nueva funcionalidad
- `fix:` Corrección de bug
- `docs:` Cambios en documentación
- `refactor:` Refactorización de código
- `test:` Agregar o modificar tests
- `chore:` Tareas de mantenimiento


## 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para más detalles.

## 📞 Soporte

Para reportar bugs o solicitar funcionalidades:

- 🐛 **Issues**: [GitHub Issues](link-to-issues)
- 📧 **Email**: support@edutin.com
- 💬 **Slack**: #aws-layers

## 🔗 Enlaces útiles

- [Documentación de AWS Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [OpenSearch Documentation](https://opensearch.org/docs/latest/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)

---

**Hecho con ❤️ por el equipo de Edutin**

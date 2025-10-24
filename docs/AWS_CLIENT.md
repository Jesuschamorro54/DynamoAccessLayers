# aws_client - Cliente AWS Services

Documentación completa del módulo `aws_client` para interacciones con servicios de AWS.

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Arquitectura](#arquitectura)
- [AWS Lambda](#aws-lambda)
- [EventBridge](#eventbridge)
- [WebSocket API](#websocket-api)
- [OpenSearch](#opensearch)
- [Mejores Prácticas](#mejores-prácticas)
- [Solución de Problemas](#solución-de-problemas)

## 🎯 Introducción

El módulo `aws_client` proporciona interfaces simplificadas para interactuar con múltiples servicios de AWS:

- ✅ **AWS Lambda**: Invocación de funciones y gestión de metadata
- ✅ **EventBridge**: Envío de eventos estructurados con trazabilidad
- ✅ **WebSocket API**: Comunicación en tiempo real
- ✅ **OpenSearch**: Búsquedas full-text y operaciones CRUD

### Ventajas

| Característica | boto3 directo | aws_client |
|----------------|---------------|------------|
| Validación de datos | Manual | Automática |
| Estructuración de eventos | Manual | Estandarizada |
| Manejo de errores | Manual | Centralizado |
| Trazabilidad | Manual | Integrada |
| Type hints | Parcial | Completo |

## 🏗️ Arquitectura

```
aws_client/
├── aws_lambda.py               # Lambda & EventBridge functions
└── open_search/
    ├── __init__.py
    ├── open_search_module.py   # Cliente OpenSearch base
    ├── op_searches.py          # Operaciones de búsqueda
    ├── op_updates.py           # Operaciones CRUD
    └── op_querys.py            # Query builders y modelos
```

---

## 🔧 AWS Lambda

Funciones para invocar otras Lambdas y obtener metadata de ejecución.

### Imports

```python
from aws_client.aws_lambda import (
    get_lambda_metadata,    # Obtener metadata de Lambda
    invoke_lambda,          # Invocar otra Lambda
    invoke_event_bus,       # Enviar evento a EventBridge
    send_webSocket_message  # Enviar mensaje WebSocket
)
```

---

### get_lambda_metadata

Extrae metadata del contexto de ejecución de Lambda.

#### Firma

```python
def get_lambda_metadata(context) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `context` | LambdaContext | Contexto de AWS Lambda | ✅ |

#### Retorno

```python
{
    "FunctionName": "mi-funcion",
    "Alias": "develop",          # "develop" si no hay alias
    "Version": "$LATEST",
    "Memory": 512,               # MB
    "Arn": "arn:aws:lambda:...",
    "RequestId": "abc-123-..."
}
```

#### Ejemplos

**Uso básico**

```python
from aws_client.aws_lambda import get_lambda_metadata

def lambda_handler(event, context):
    # Obtener metadata
    metadata = get_lambda_metadata(context)
    
    print(f"Función: {metadata['FunctionName']}")
    print(f"Memoria: {metadata['Memory']} MB")
    print(f"Request ID: {metadata['RequestId']}")
    
    return {
        'statusCode': 200,
        'body': json.dumps(metadata)
    }
```

**Para trazabilidad**

```python
import logging
import json

logger = logging.getLogger()

def lambda_handler(event, context):
    metadata = get_lambda_metadata(context)
    
    # Log estructurado con metadata
    logger.info(json.dumps({
        'action': 'function_invoked',
        'function': metadata['FunctionName'],
        'request_id': metadata['RequestId'],
        'version': metadata['Version']
    }))
    
    # Tu código aquí...
```

**En combinación con otras funciones**

```python
from aws_client.aws_lambda import get_lambda_metadata, invoke_lambda

def lambda_handler(event, context):
    metadata = get_lambda_metadata(context)
    
    # Invocar otra Lambda usando el mismo alias
    result = invoke_lambda(
        lambda_name='otra-funcion',
        payload={'data': 'example'},
        qualifier=metadata['Alias']  # Usar el mismo ambiente
    )
    
    return result
```

---

### invoke_lambda

Invoca otra función Lambda de forma asíncrona o síncrona.

#### Firma

```python
def invoke_lambda(
    lambda_name: str,
    payload: dict,
    invocation_type: str = "Event",
    qualifier: str = 'develop',
    log_enabled: bool = True
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido | Default |
|-----------|------|-------------|-----------|---------|
| `lambda_name` | str | Nombre de la función Lambda | ✅ | - |
| `payload` | dict | Datos a enviar | ✅ | - |
| `invocation_type` | str | "Event" (async) o "RequestResponse" (sync) | ❌ | "Event" |
| `qualifier` | str | Alias o versión | ❌ | "develop" |
| `log_enabled` | bool | Habilitar logging | ❌ | True |

#### Retorno

- **Async ("Event")**: `{}`
- **Sync ("RequestResponse")**: Response object de la Lambda invocada

#### Ejemplos

**Invocación asíncrona (fire and forget)**

```python
from aws_client.aws_lambda import invoke_lambda

def lambda_handler(event, context):
    # Invocar otra Lambda de forma asíncrona
    payload = {
        'user_id': '12345',
        'action': 'send_email',
        'email': 'user@example.com'
    }
    
    invoke_lambda(
        lambda_name='email-sender',
        payload=payload,
        invocation_type='Event'  # Asíncrona
    )
    
    # Continúa sin esperar respuesta
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Email en proceso'})
    }
```

**Invocación síncrona (espera respuesta)**

```python
def lambda_handler(event, context):
    # Invocar Lambda y esperar resultado
    payload = {
        'user_id': '12345',
        'calculate': 'statistics'
    }
    
    response = invoke_lambda(
        lambda_name='data-processor',
        payload=payload,
        invocation_type='RequestResponse'  # Síncrona
    )
    
    # Procesar respuesta
    if response['StatusCode'] == 200:
        result = json.loads(response['Payload'].read())
        print(f"Resultado: {result}")
    
    return result
```

**Usando diferentes ambientes**

```python
from aws_client.aws_lambda import get_lambda_metadata, invoke_lambda

def lambda_handler(event, context):
    metadata = get_lambda_metadata(context)
    current_env = metadata['Alias']
    
    # Invocar en el mismo ambiente
    invoke_lambda(
        lambda_name='worker-function',
        payload={'task': 'process'},
        qualifier=current_env  # develop, staging, production
    )
```

**Con manejo de errores**

```python
def lambda_handler(event, context):
    try:
        response = invoke_lambda(
            lambda_name='processor',
            payload={'data': event},
            invocation_type='RequestResponse',
            log_enabled=True
        )
        
        if response['StatusCode'] != 200:
            raise Exception(f"Lambda error: {response['StatusCode']}")
        
        result = json.loads(response['Payload'].read())
        return result
        
    except Exception as e:
        logger.error(f"Error invocando Lambda: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

---

## 📡 EventBridge

Envío de eventos estructurados a AWS EventBridge.

### invoke_event_bus

Envía un evento a EventBridge con estructura estandarizada.

#### Firma

```python
def invoke_event_bus(
    EventBusName: str,
    Source: str,
    payload: dict,
    DetailType: str,
    lambda_details: dict,
    JSONSTRINGIFY: bool = True,
    log_enabled: bool = False
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido | Default |
|-----------|------|-------------|-----------|---------|
| `EventBusName` | str | Nombre del event bus | ✅ | - |
| `Source` | str | Identificador de origen | ✅ | - |
| `payload` | dict | Datos del evento | ✅ | - |
| `DetailType` | str | Tipo de evento | ✅ | - |
| `lambda_details` | dict | Metadata de Lambda origen | ✅ | - |
| `JSONSTRINGIFY` | bool | Serializar body a JSON | ❌ | True |
| `log_enabled` | bool | Habilitar logging | ❌ | False |

#### Retorno

```python
{
    "status": True,            # True si fue exitoso
    "FailedEntryCount": 0      # Número de entradas fallidas
}
```

#### Estructura del Evento

El evento enviado tiene la siguiente estructura:

```python
{
    "EventBridgeTracer": "EventBridge:{bus}:Source:{source}:DetailType:{type}:LambdaOrigin:{function}",
    "timeEpoch": 1234567890123,
    "origin": {
        "resource": "Lambda",
        "arn": "arn:aws:lambda:...",
        "version": "$LATEST",
        "alias": "develop",
        "RequestId": "abc-123",
        "FunctionName": "mi-funcion"
    },
    "LambdaOriginEvent": {...},  # Evento original de Lambda
    "authorizer": {...},         # Claims del authorizer
    "params": {...},             # Query/path parameters
    "body": {                    # Payload principal
        "data": {...}
    }
}
```

#### Ejemplos

**Evento básico**

```python
from aws_client.aws_lambda import get_lambda_metadata, invoke_event_bus

def lambda_handler(event, context):
    # Obtener metadata
    lambda_meta = get_lambda_metadata(context)
    
    # Preparar payload
    payload = {
        'user_id': '12345',
        'action': 'purchase_course',
        'course_id': '6789',
        'amount': 99.99
    }
    
    # Enviar evento
    result = invoke_event_bus(
        EventBusName='edutin-events',
        Source='course.purchases',
        payload=payload,
        DetailType='CoursePurchase',
        lambda_details=lambda_meta
    )
    
    if result['status']:
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Evento enviado'})
        }
    else:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Falló envío de evento'})
        }
```

**Con contexto de API Gateway**

```python
def lambda_handler(event, context):
    lambda_meta = get_lambda_metadata(context)
    
    # Extraer contexto de API Gateway
    authorizer = event.get('requestContext', {}).get('authorizer', {})
    query_params = event.get('queryStringParameters', {})
    
    # Payload con contexto
    payload = {
        'user_id': authorizer.get('jwt', {}).get('claims', {}).get('id'),
        'action': 'update_profile',
        'changes': json.loads(event.get('body', '{}')),
        # Estos campos se extraen automáticamente:
        'lambda_event': event,           # → LambdaOriginEvent
        'authorizer': authorizer,        # → authorizer
        'params': query_params           # → params
    }
    
    result = invoke_event_bus(
        EventBusName='edutin-events',
        Source='user.profile',
        payload=payload,
        DetailType='ProfileUpdate',
        lambda_details=lambda_meta,
        log_enabled=True  # Ver detalles en CloudWatch
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({'event_sent': result['status']})
    }
```

**Evento sin JSON stringify**

```python
# Útil cuando el consumer espera un objeto, no string
payload = {
    'data': {
        'nested': {
            'structure': 'value'
        }
    }
}

result = invoke_event_bus(
    EventBusName='edutin-events',
    Source='data.sync',
    payload=payload,
    DetailType='DataSync',
    lambda_details=lambda_meta,
    JSONSTRINGIFY=False  # body será un objeto, no string
)
```

---

## 🔍 OpenSearch

Cliente completo para búsquedas full-text y operaciones CRUD en OpenSearch.

### Configuración

Variables de entorno requeridas:

```bash
OPENSEARCH_HOST=your-opensearch-endpoint.com
OPENSEARCH_PORT=9200
OPENSEARCH_USERNAME=admin
OPENSEARCH_PASSWORD=your-secure-password
OPENSEARCH_KEY_ACCESS=your-api-key
```

### Imports

```python
# Búsquedas
from aws_client.open_search.op_searches import op_search

# Operaciones CRUD
from aws_client.open_search.op_updates import (
    op_create,       # Crear documento
    op_update,       # Actualizar documento
    op_delete,       # Eliminar documento
    op_bulk_create   # Crear múltiples documentos
)
```

---

### op_search

Realiza búsquedas full-text en índices de OpenSearch.

#### Firma

```python
def op_search(
    index_name: str,
    query: str,
    last_item: list = None,
    **kwargs
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido | Default |
|-----------|------|-------------|-----------|---------|
| `index_name` | str | Nombre del índice ('courses' o 'dev-users-v3') | ✅ | - |
| `query` | str | Término de búsqueda | ✅ | - |
| `last_item` | list | Cursor para paginación | ❌ | None |
| `size` | int | Número de resultados | ❌ | 10 |
| `raw_data` | bool | Retornar datos sin procesar | ❌ | False |
| `name_boost` | int | Boost para campo name | ❌ | 33 |
| `description_boost` | int | Boost para description | ❌ | 2 |
| `score` | int | Score mínimo | ❌ | 5 |
| `log_query` | bool | Log de query construido | ❌ | False |

#### Índices soportados

**courses**
- Búsqueda de cursos y rutas de aprendizaje
- Filtrado por privacidad y estado
- Sorting por relevancia y estudiantes

**dev-users-v3**
- Búsqueda de usuarios
- Soporte para roles y permisos
- Búsqueda predictiva (search-as-you-type)

#### Retorno

**Para courses:**
```python
{
    "courses": [...],        # Lista de cursos/rutas
    "last_item": [...],      # Cursor para paginación
    "total_hits": 150        # Total encontrados
}
```

**Para users:**
```python
{
    "users": [...],          # Lista de usuarios
    "last_item": [...],      # Cursor para paginación
    "total_hits": 42         # Total encontrados
}
```

#### Ejemplos

**Búsqueda simple de cursos**

```python
from aws_client.open_search.op_searches import op_search

def lambda_handler(event, context):
    search_term = event.get('queryStringParameters', {}).get('q', '')
    
    # Buscar cursos
    result = op_search(
        index_name='courses',
        query=search_term,
        size=20
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'total': result['total_hits'],
            'courses': result['courses']
        })
    }
```

**Búsqueda con paginación**

```python
def search_courses_paginated(query: str, page_size: int = 10):
    """Búsqueda con paginación manual"""
    all_courses = []
    last_item = None
    max_pages = 5
    
    for page in range(max_pages):
        result = op_search(
            index_name='courses',
            query=query,
            last_item=last_item,
            size=page_size
        )
        
        all_courses.extend(result['courses'])
        
        # Si no hay más resultados, terminar
        if not result.get('last_item'):
            break
        
        last_item = result['last_item']
    
    return all_courses
```

**Búsqueda con boost personalizado**

```python
# Dar más peso al nombre del curso
result = op_search(
    index_name='courses',
    query='python',
    name_boost=50,           # Más relevancia en nombre
    description_boost=5,     # Menos relevancia en descripción
    score=10                 # Score mínimo más alto
)
```

**Búsqueda de usuarios con privilegios**

```python
def search_users(query: str, user_context: dict):
    """Búsqueda de usuarios según contexto"""
    
    result = op_search(
        index_name='dev-users-v3',
        query=query,
        size=20,
        # Parámetros específicos para usuarios
        role=user_context.get('role'),
        business_id=user_context.get('business_id'),
        business_role=user_context.get('business_role'),
        privileged_roles={'admin', 'supervisor', 'manager'}
    )
    
    return result['users']
```

**Búsqueda con highlighting**

```python
result = op_search(
    index_name='courses',
    query='machine learning',
    size=10
)

# Los resultados incluyen highlights
for course in result['courses']:
    if '_highlights' in course:
        print(f"Nombre resaltado: {course['_highlights']['name']}")
```

**Raw data para análisis**

```python
# Obtener datos sin procesar de OpenSearch
result = op_search(
    index_name='courses',
    query='python',
    raw_data=True,
    log_query=True  # Ver query construido
)

# Analizar estructura
for hit in result['courses']:
    print(f"Score: {hit['_score']}")
    print(f"Source: {hit['_source']}")
```

---

### op_create

Crea un nuevo documento en OpenSearch.

#### Firma

```python
def op_create(
    index_name: str,
    document: dict,
    doc_id: str = None
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `index_name` | str | Nombre del índice | ✅ |
| `document` | dict | Datos del documento | ✅ |
| `doc_id` | str | ID personalizado (opcional) | ❌ |

#### Retorno

```python
{
    "status": True,
    "message": "Document created successfully",
    "operation": "create",
    "index": "courses",
    "document_id": "1234",
    "result": {...},
    "document": {...}
}
```

#### Ejemplos

**Crear curso**

```python
from aws_client.open_search.op_updates import op_create

def lambda_handler(event, context):
    course_data = {
        'course_id': '1234',
        'slug': 'python-basics',
        'name': 'Python Básico',
        'students': 0,
        'rating': None,
        'description': 'Curso de introducción a Python',
        'is_route': False,
        'status': 1,
        'privacy': 'public'
    }
    
    result = op_create(
        index_name='courses',
        document=course_data
    )
    
    if result['status']:
        return {
            'statusCode': 201,
            'body': json.dumps({
                'message': 'Curso creado',
                'id': result['document_id']
            })
        }
    else:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': result['error']})
        }
```

**Normalización automática**

```python
# Los campos de búsqueda se normalizan automáticamente
course_data = {
    'course_id': '1234',
    'name': 'Introducción á Pythón',  # Con tildes
    'slug': 'Introducción-Python',
    # ...
}

result = op_create('courses', course_data)

# Se crean automáticamente:
# - name_search: "introduccion a python" (normalizado)
# - slug_search: "introduccion-python" (normalizado)
```

---

### op_update

Actualiza un documento existente en OpenSearch.

#### Firma

```python
def op_update(
    index_name: str,
    document: dict,
    doc_id: str = None
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `index_name` | str | Nombre del índice | ✅ |
| `document` | dict | Campos a actualizar | ✅ |
| `doc_id` | str | ID del documento (opcional si está en document) | ❌ |

#### Retorno

```python
{
    "status": True,
    "message": "Document updated successfully",
    "operation": "update",
    "index": "courses",
    "document_id": "1234",
    "result": {...},
    "document": {...}
}
```

#### Ejemplos

**Actualización parcial**

```python
from aws_client.open_search.op_updates import op_update

# Solo actualizar campos específicos
update_data = {
    'course_id': '1234',
    'students': 5000,
    'rating': 4.8
}

result = op_update(
    index_name='courses',
    document=update_data
)
```

**Actualización con normalización**

```python
# Actualizar nombre (se normaliza automáticamente)
update_data = {
    'course_id': '1234',
    'name': 'Nuevo Título del Curso',
    'description': 'Nueva descripción con <strong>HTML</strong>'
}

result = op_update('courses', update_data)
# name_search y description se actualizan automáticamente
```

---

### op_delete

Elimina un documento de OpenSearch.

#### Firma

```python
def op_delete(
    index_name: str,
    doc_id: str
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `index_name` | str | Nombre del índice | ✅ |
| `doc_id` | str | ID del documento | ✅ |

#### Retorno

```python
{
    "status": True,
    "message": "Document deleted successfully",
    "operation": "delete",
    "index": "courses",
    "document_id": "1234",
    "result": {...}
}
```

#### Ejemplos

```python
from aws_client.open_search.op_updates import op_delete

def lambda_handler(event, context):
    course_id = event['pathParameters']['id']
    
    result = op_delete(
        index_name='courses',
        doc_id=course_id
    )
    
    if result['status']:
        return {
            'statusCode': 200,
            'body': json.dumps({'message': 'Curso eliminado'})
        }
    else:
        return {
            'statusCode': 404,
            'body': json.dumps({'error': 'Curso no encontrado'})
        }
```

---

### op_bulk_create

Crea múltiples documentos en una sola operación.

#### Firma

```python
def op_bulk_create(
    documents: list,
    index_name: str
) -> dict
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `documents` | list | Lista de documentos | ✅ |
| `index_name` | str | Nombre del índice | ✅ |

#### Retorno

```python
{
    "status": True,
    "message": "Bulk create completed: 25 successful, 0 failed",
    "operation": "bulk_create",
    "index": "courses",
    "total_documents": 25,
    "successful": 25,
    "failed": 0,
    "errors": None,
    "result": {...}
}
```

#### Ejemplos

```python
from aws_client.open_search.op_updates import op_bulk_create

def lambda_handler(event, context):
    courses = [
        {
            'course_id': '1',
            'name': 'Curso 1',
            'slug': 'curso-1',
            'students': 100,
            'is_route': False,
            'status': 1
        },
        {
            'course_id': '2',
            'name': 'Curso 2',
            'slug': 'curso-2',
            'students': 200,
            'is_route': False,
            'status': 1
        }
    ]
    
    result = op_bulk_create(
        documents=courses,
        index_name='courses'
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'successful': result['successful'],
            'failed': result['failed']
        })
    }
```

---

## 🐛 Solución de Problemas

### Lambda invocation timeout

**Problema**: Timeout al invocar Lambda síncrona.

**Solución**:
```python
# Usar invocación asíncrona para tareas largas
invoke_lambda(
    lambda_name='long-task',
    payload=data,
    invocation_type='Event'  # No esperar respuesta
)

# O aumentar timeout de Lambda en AWS Console
```

### EventBridge events not received

**Problema**: Eventos no llegan a destino.

**Verificar**:
1. Event bus existe
2. Reglas de EventBridge configuradas
3. Permisos IAM correctos

```python
# Debug con logging
result = invoke_event_bus(
    EventBusName='edutin-events',
    Source='test',
    payload={'test': 'data'},
    DetailType='Test',
    lambda_details=metadata,
    log_enabled=True  # ← Ver detalles
)

print(f"Failed entries: {result['FailedEntryCount']}")
```

### OpenSearch connection failed

**Problema**: No puede conectar a OpenSearch.

**Verificar**:
```python
# 1. Variables de entorno
import os

print(f"Host: {os.getenv('OPENSEARCH_HOST')}")
print(f"Port: {os.getenv('OPENSEARCH_PORT')}")
print(f"Username: {os.getenv('OPENSEARCH_USERNAME')}")
# No imprimir password en producción

# 2. Permisos de seguridad
# Verificar security groups y VPC

# 3. Probar conexión básica
from aws_client.open_search.open_search_module import OpenSearchClient

try:
    client = OpenSearchClient()
    print("Conexión exitosa")
except Exception as e:
    print(f"Error: {str(e)}")
```

### OpenSearch indexing errors

**Problema**: Documentos no se indexan correctamente.

**Solución**:
```python
# Validar datos antes de indexar
from aws_client.open_search.op_updates import op_create

def safe_index_document(document: dict, index: str):
    # Validar campos requeridos
    required_fields = ['course_id', 'name', 'slug']
    
    for field in required_fields:
        if field not in document or not document[field]:
            logger.error(f"Missing required field: {field}")
            return {'status': False, 'error': f'Missing {field}'}
    
    try:
        result = op_create(index_name=index, document=document)
        return result
    except Exception as e:
        logger.error(f"Indexing error: {str(e)}", exc_info=True)
        return {'status': False, 'error': str(e)}
```

---

## 📚 Referencias

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [EventBridge User Guide](https://docs.aws.amazon.com/eventbridge/latest/userguide/)
- [API Gateway WebSocket APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-api.html)
- [OpenSearch Documentation](https://opensearch.org/docs/latest/)

---

**[⬅️ Volver al README principal](../README.md)**


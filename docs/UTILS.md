# utils - Utilidades Generales

Documentación del módulo `utils` con utilidades para Lambda functions.

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Logger Runtime](#logger-runtime)
- [Ejemplos de Uso](#ejemplos-de-uso)
- [Mejores Prácticas](#mejores-prácticas)

## 🎯 Introducción

El módulo `utils` proporciona utilidades generales para simplificar tareas comunes en funciones Lambda, principalmente enfocadas en logging estructurado y captura de contexto.

### Características

- ✅ **Logging estructurado**: Captura automática de contexto de ejecución
- ✅ **Múltiples fuentes**: Soporte para API Gateway, SQS, EventBridge
- ✅ **JSON formatting**: Logs en formato JSON para fácil parsing
- ✅ **Información de usuario**: Extracción automática de claims de authorizer
- ✅ **Trazabilidad**: Contexto completo para debugging

## 📝 Logger Runtime

### logger_runtime_event

Captura y registra automáticamente el contexto completo de un evento Lambda.

#### Firma

```python
def logger_runtime_event(event: dict) -> None
```

#### Parámetros

| Parámetro | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `event` | dict | Evento de Lambda | ✅ |

#### Información Capturada

**API Gateway HTTP/REST:**
```python
{
    "origin": "https://app.edutin.com",
    "device": "Mozilla/5.0...",
    "platform": "Windows",
    "lambda_stage": "develop",
    "api_stage": "v1",
    "raw_route": "POST /api/courses",
    "domain": "api.edutin.com",
    "path": "/api/courses",
    "query_string": {"filter": "active"},
    "path_parameters": {"id": "123"},
    "user_id": "425034",
    "user_email": "user@example.com",
    "user_role": "student",
    "business_id": "abc-123",
    "business_role": "owner",
    "body": {...}
}
```

**SQS:**
```python
{
    "attributes": {
        "ApproximateReceiveCount": "1",
        "SentTimestamp": "1234567890"
    },
    "messageAttributes": {...},
    "body": {...}
}
```

**EventBridge:**
```python
{
    "eventBridgeTracer": "EventBridge:edutin-events:Source:...",
    "origins": {
        "resource": "Lambda",
        "arn": "arn:aws:lambda:...",
        "FunctionName": "origin-function"
    },
    "source": "course.purchases",
    "resource": ["arn:aws:events:..."],
    "body": {...}
}
```

---

## 💡 Ejemplos de Uso

### Uso Básico

```python
from utils.logs import logger_runtime_event
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    # Log completo del evento con contexto
    logger_runtime_event(event)
    
    # Tu código aquí...
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Success'})
    }
```

**Output en CloudWatch:**
```json
{
    "origin": "https://app.edutin.com",
    "device": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
    "lambda_stage": "develop",
    "api_stage": "v1",
    "path": "/api/courses/search",
    "query_string": {"q": "python", "limit": "20"},
    "user_id": "425034",
    "user_email": "user@edutin.com",
    "user_role": "student",
    "body": {
        "filters": {
            "category": "programming"
        }
    }
}
```

### Con API Gateway

```python
def lambda_handler(event, context):
    # Capturar evento HTTP
    logger_runtime_event(event)
    
    # Extraer información del log
    path = event.get('rawPath', '')
    method = event.get('requestContext', {}).get('http', {}).get('method', '')
    user_id = event.get('requestContext', {}).get('authorizer', {}).get('jwt', {}).get('claims', {}).get('id')
    
    logger.info(f"Request: {method} {path} by user {user_id}")
    
    # Procesar request...
    
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps({'message': 'Processed'})
    }
```

### Con SQS

```python
def lambda_handler(event, context):
    # Log de evento SQS
    logger_runtime_event(event)
    
    # Procesar mensajes
    for record in event.get('Records', []):
        message_body = json.loads(record['body'])
        
        logger.info(f"Processing message: {record['messageId']}")
        
        # Procesar mensaje...
        process_message(message_body)
    
    return {
        'statusCode': 200,
        'body': json.dumps({'processed': len(event.get('Records', []))})
    }
```

### Con EventBridge

```python
def lambda_handler(event, context):
    # Log de evento EventBridge
    logger_runtime_event(event)
    
    # Extraer detalles
    detail = event.get('detail', {})
    source = event.get('source', '')
    
    logger.info(f"Received event from {source}")
    
    # Procesar según la fuente
    if source == 'course.purchases':
        handle_purchase(detail)
    elif source == 'user.updates':
        handle_user_update(detail)
    
    return {'statusCode': 200}
```

### Logging Avanzado con Contexto

```python
from utils.logs import logger_runtime_event
import logging
import json
from datetime import datetime

logger = logging.getLogger()

def lambda_handler(event, context):
    # Capturar evento completo
    logger_runtime_event(event)
    
    # Log estructurado de inicio
    start_time = datetime.utcnow()
    request_id = context.aws_request_id
    
    logger.info(json.dumps({
        'stage': 'start',
        'request_id': request_id,
        'timestamp': start_time.isoformat(),
        'function': context.function_name
    }))
    
    try:
        # Tu código aquí...
        result = process_request(event)
        
        # Log de éxito
        logger.info(json.dumps({
            'stage': 'success',
            'request_id': request_id,
            'duration_ms': (datetime.utcnow() - start_time).total_seconds() * 1000,
            'result_count': len(result)
        }))
        
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
        
    except Exception as e:
        # Log de error
        logger.error(json.dumps({
            'stage': 'error',
            'request_id': request_id,
            'error_type': type(e).__name__,
            'error_message': str(e),
            'duration_ms': (datetime.utcnow() - start_time).total_seconds() * 1000
        }))
        
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```
```
# Usar en Lambda
def lambda_handler(event, context):
    logger_runtime_event(event)
    
    # Publicar métricas
    publish_metric('RequestsReceived', 1)
    
    try:
        result = process(event)
        publish_metric('SuccessfulRequests', 1)
        return result
    except Exception as e:
        publish_metric('FailedRequests', 1)
        raise
```

---

## 🔗 Referencias
- [Lambda Logging](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html)

---

**[⬅️ Volver al README principal](../README.md)**


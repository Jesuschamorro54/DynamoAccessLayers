# Índice de Documentación

Bienvenido a la documentación completa de **AWS Edutin Layer**.

## 📚 Documentación Principal

### [README Principal](../README.md)
Punto de entrada al proyecto con descripción general, instalación rápida y ejemplos básicos.

**Incluye:**
- Descripción del proyecto
- Características principales
- Arquitectura general
- Instalación
- Inicio rápido
- Ejemplos de uso
- Mejores prácticas

---

## 🔍 Documentación de Módulos

### [ddb_client - Cliente DynamoDB](DDB_CLIENT.md)
Documentación completa del módulo para operaciones con AWS DynamoDB.

**Contenido:**
- Configuración del módulo
- Funciones de búsqueda:
  - `dynamo_search` - Búsquedas avanzadas
  - `dynamo_counter` - Contador de registros
  - `batch_get_items` - Operaciones batch
- Funciones de actualización:
  - `dynamo_create` - Crear registros
  - `dynamo_update` - Actualizar registros
  - `dynamo_increase` - Incrementos atómicos
  - `dynamo_delete` - Eliminar registros
  - `batch_create_items` - Crear múltiples registros
  - `batch_delete_items` - Eliminar múltiples registros
- Helpers de comparación:
  - `DynamoComparison` - Operadores avanzados
  - Operadores lógicos (OR, AND)
- Ejemplos exhaustivos
- Solución de problemas

**Nivel:** Intermedio - Avanzado  
**Tiempo de lectura:** ~45 minutos

---

### [aws_client - Cliente AWS Services](AWS_CLIENT.md)
Documentación del módulo para interacciones con servicios de AWS.

**Contenido:**
- **AWS Lambda:**
  - `get_lambda_metadata` - Obtener metadata
  - `invoke_lambda` - Invocar funciones
- **EventBridge:**
  - `invoke_event_bus` - Enviar eventos
  - Estructura de eventos
  - Trazabilidad
- **WebSocket API:**
  - `send_webSocket_message` - Mensajes en tiempo real
  - Manejo de conexiones
- **OpenSearch:**
  - `op_search` - Búsquedas full-text
  - `op_create` - Crear documentos
  - `op_update` - Actualizar documentos
  - `op_delete` - Eliminar documentos
  - `op_bulk_create` - Operaciones masivas
  - Normalización de texto
- Ejemplos prácticos
- Solución de problemas

**Nivel:** Intermedio - Avanzado  
**Tiempo de lectura:** ~60 minutos

---

### [utils - Utilidades Generales](UTILS.md)
Documentación de utilidades para Lambda functions.

**Contenido:**
- `logger_runtime_event` - Logging estructurado
- Captura de contexto automática
- Integración con CloudWatch
- CloudWatch Insights queries
- Métricas personalizadas
- Ejemplos de uso
- Mejores prácticas

**Nivel:** Básico - Intermedio  
**Tiempo de lectura:** ~20 minutos

---

## 🤝 Guías de Desarrollo

### [Guía de Contribución](CONTRIBUTING.md)
Todo lo que necesitas saber para contribuir al proyecto.

**Contenido:**
- Código de conducta
- Cómo contribuir
- Configuración del entorno
- Estándares de código
- Testing
- Proceso de Pull Request
- Deployment
- Versionado

**Nivel:** Todos los niveles  
**Tiempo de lectura:** ~30 minutos

---

### [Limpieza de Lambda Layers](CLEANUP_LAYERS.md)
Guía para eliminar versiones antiguas de Lambda Layers de forma segura.

**Contenido:**
- Uso del script cleanup_layers.py
- Verificación de uso en funciones Lambda
- Protección de versiones recientes
- Modo dry-run
- Ejemplos prácticos
- Estrategias de limpieza
- Troubleshooting

**Nivel:** Básico - Intermedio  
**Tiempo de lectura:** ~15 minutos

---

## 📋 Recursos Adicionales

### [CHANGELOG](../CHANGELOG.md)
Historial de cambios y versiones del proyecto.

**Contenido:**
- Cambios por versión
- Features agregados
- Bugs corregidos
- Breaking changes
- Deprecaciones

**Actualización:** Continua

---

### [LICENSE](../LICENSE)
Licencia MIT del proyecto.

---

## 🚀 Rutas de Aprendizaje

### Para Principiantes

1. **Inicio Rápido** (15 min)
   - Lee el [README Principal](../README.md)
   - Ejecuta los ejemplos de inicio rápido
   - Familiarízate con la estructura

2. **Primeros Pasos** (30 min)
   - Configuración básica en `config.py`
   - Ejemplos simples de [ddb_client](DDB_CLIENT.md#ejemplos-de-uso)
   - Prueba búsquedas básicas

3. **Logging** (20 min)
   - Implementa [logger_runtime_event](UTILS.md)
   - Revisa logs en CloudWatch
   - Experimenta con logs estructurados

### Para Desarrolladores Intermedios

1. **DynamoDB Avanzado** (45 min)
   - Estudia [DynamoComparison](DDB_CLIENT.md#helpers)
   - Implementa operaciones batch
   - Usa operadores lógicos (OR, AND)

2. **Servicios AWS** (60 min)
   - Aprende [aws_client](AWS_CLIENT.md)
   - Implementa invocaciones Lambda
   - Envía eventos a EventBridge

3. **OpenSearch** (45 min)
   - Configura [OpenSearch](AWS_CLIENT.md#opensearch)
   - Implementa búsquedas full-text
   - Sincroniza con DynamoDB

### Para Contribuidores

1. **Setup de Desarrollo** (30 min)
   - Configura entorno local
   - Instala pre-commit hooks
   - Ejecuta tests

2. **Estándares** (45 min)
   - Lee [CONTRIBUTING.md](CONTRIBUTING.md)
   - Revisa estándares de código
   - Entiende el flujo de PR

3. **Primera Contribución** (Variable)
   - Encuentra un issue "good first issue"
   - Implementa y testea
   - Crea tu primer PR

---

## 🔗 Enlaces Rápidos

### Operaciones Comunes

| Operación | Documentación |
|-----------|---------------|
| Buscar en DynamoDB | [dynamo_search](DDB_CLIENT.md#dynamo_search) |
| Crear registro | [dynamo_create](DDB_CLIENT.md#dynamo_create) |
| Actualizar registro | [dynamo_update](DDB_CLIENT.md#dynamo_update) |
| Eliminar registro | [dynamo_delete](DDB_CLIENT.md#dynamo_delete) |
| Invocar Lambda | [invoke_lambda](AWS_CLIENT.md#invoke_lambda) |
| Enviar evento | [invoke_event_bus](AWS_CLIENT.md#invoke_event_bus) |
| Buscar en OpenSearch | [op_search](AWS_CLIENT.md#op_search) |
| Logging | [logger_runtime_event](UTILS.md#logger_runtime_event) |

### Por Casos de Uso

| Caso de Uso | Ejemplo |
|-------------|---------|
| Búsqueda con filtros | [Ver ejemplo](DDB_CLIENT.md#búsqueda-con-filtros) |
| Paginación | [Ver ejemplo](DDB_CLIENT.md#paginación) |
| Operaciones batch | [Ver ejemplo](DDB_CLIENT.md#operaciones-batch) |
| Incrementos con límites | [Ver ejemplo](DDB_CLIENT.md#incrementos-con-límites) |
| Eventos con trazabilidad | [Ver ejemplo](AWS_CLIENT.md#patrón-de-trazabilidad) |
| WebSocket broadcast | [Ver ejemplo](AWS_CLIENT.md#broadcast-a-múltiples-conexiones) |
| Búsqueda full-text | [Ver ejemplo](AWS_CLIENT.md#búsqueda-simple-de-cursos) |
| Limpieza de layers | [Ver guía](CLEANUP_LAYERS.md) |

---

## 🎯 Objetivos de Aprendizaje

### Nivel Básico
- [ ] Entender la estructura del proyecto
- [ ] Configurar el módulo correctamente
- [ ] Realizar búsquedas simples en DynamoDB
- [ ] Implementar logging básico
- [ ] Crear y actualizar registros

### Nivel Intermedio
- [ ] Usar helpers de comparación avanzados
- [ ] Implementar operaciones batch eficientes
- [ ] Invocar otras funciones Lambda
- [ ] Enviar eventos a EventBridge
- [ ] Manejar paginación correctamente

### Nivel Avanzado
- [ ] Optimizar queries de DynamoDB
- [ ] Implementar búsquedas complejas en OpenSearch
- [ ] Sincronizar DynamoDB con OpenSearch
- [ ] Implementar WebSocket en tiempo real
- [ ] Contribuir al proyecto

---

## 📞 Soporte

### ¿Necesitas Ayuda?

1. **Documentación**: Busca en este índice
2. **Ejemplos**: Revisa los ejemplos de código
3. **Issues**: [GitHub Issues](https://github.com/Jesuschamorro54/DynamoAccessLayers/issues)
5. **Email**: dev@edutin.com

### Recursos Externos

- [AWS DynamoDB Docs](https://docs.aws.amazon.com/dynamodb/)
- [AWS Lambda Docs](https://docs.aws.amazon.com/lambda/)
- [OpenSearch Docs](https://opensearch.org/docs/)
- [Boto3 Docs](https://boto3.amazonaws.com/v1/documentation/)

---

## 🔄 Actualizaciones

Esta documentación se actualiza continuamente. Última actualización: 23 Octubre 2025

Para estar al día:
- ⭐ Sigue el repositorio en GitHub
- 📬 Suscríbete a las releases

---

**¿Listo para empezar? → [README Principal](../README.md)**


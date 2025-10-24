# Limpieza de Lambda Layers

Guía completa para usar el script `cleanup_layers.py` que permite eliminar versiones de Lambda Layers de manera segura.

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Características](#características)
- [Requisitos](#requisitos)
- [Uso](#uso)
- [Ejemplos](#ejemplos)
- [Opciones Avanzadas](#opciones-avanzadas)
- [Seguridad](#seguridad)
- [Troubleshooting](#troubleshooting)

## 🎯 Descripción

El script `cleanup_layers.py` permite eliminar versiones antiguas de Lambda Layers de manera masiva, con verificaciones de seguridad integradas:

- ✅ Verifica si hay Lambdas usando cada versión antes de eliminar
- ✅ Protege automáticamente las versiones más recientes
- ✅ Soporte para rangos de versiones
- ✅ Modo dry-run para simular eliminaciones
- ✅ Confirmación de seguridad antes de eliminar

## ✨ Características

### Verificaciones de Seguridad

1. **Validación de uso**: Verifica todas las funciones Lambda que usan cada versión
2. **Protección automática**: Las últimas 3 versiones están protegidas por defecto
3. **Confirmación requerida**: Requiere confirmación explícita antes de eliminar
4. **Modo dry-run**: Simula la operación sin ejecutarla

### Flexibilidad

- Especificar versiones individuales: `1,2,3`
- Especificar rangos: `1-10`
- Combinar ambos: `1-5,8,10-15`
- Listar todas las versiones con su estado

## 📦 Requisitos

```bash
# Python 3.10+
python --version

# AWS CLI configurado
aws configure list

# Boto3 instalado
pip install boto3
```

### Permisos IAM Necesarios

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "lambda:ListLayerVersions",
                "lambda:DeleteLayerVersion",
                "lambda:ListFunctions",
                "lambda:GetFunction"
            ],
            "Resource": "*"
        }
    ]
}
```

## 🚀 Uso

### Sintaxis Básica

```bash
python cleanup_layers.py --env {develop|production} [opciones]
```

### Argumentos Principales

| Argumento | Tipo | Descripción | Requerido |
|-----------|------|-------------|-----------|
| `--env` | string | Ambiente (develop/production) | ✅ |
| `--list` | flag | Listar todas las versiones | ❌ |
| `--versions` | string | Versiones a eliminar | ❌ |
| `--dry-run` | flag | Simular sin eliminar | ❌ |
| `--force` | flag | Eliminar sin confirmación | ❌ |
| `--keep-last` | int | Versiones recientes a proteger (default: 3) | ❌ |

---

## 💡 Ejemplos

### 1. Listar Todas las Versiones

```bash
python cleanup_layers.py --env develop --list
```

**Salida:**
```
🧹 Limpieza de Lambda Layer: EdutinLayerDevelop
🌍 Ambiente: Development
======================================================================

📦 Obteniendo versiones de la Layer...
✅ Se encontraron 25 versiones

======================================================================
📋 LISTADO DE VERSIONES

Versión   1 | ⚪ Disponible | Creado: 2024-01-01T10:00:00
            Descripción: Initial version

Versión   2 | ⚪ Disponible | Creado: 2024-01-05T14:30:00
            Descripción: Added search functions

Versión   3 | 🟢 EN USO | Creado: 2024-01-10T09:15:00
            Descripción: Bug fixes
            Usado por 2 función(es):
              - api-gateway-handler
              - data-processor

Versión  23 | ⚪ Disponible | Creado: 2024-10-20T16:45:00
            Descripción: Latest features

Versión  24 | 🟢 EN USO | Creado: 2024-10-22T11:20:00
            Descripción: Performance improvements
            Usado por 5 función(es):
              - user-service
              - course-service
              - payment-service
              - auth-service
              - notification-service

Versión  25 | 🟢 EN USO | Creado: 2024-10-23T10:00:00
            Descripción: Latest deployment
            Usado por 8 función(es):
              - api-v2-handler
              - websocket-handler
              ... y 6 más

======================================================================
```

### 2. Eliminar Versiones Específicas

```bash
python cleanup_layers.py --env develop --versions "1,2,3"
```

### 3. Eliminar un Rango de Versiones

```bash
python cleanup_layers.py --env develop --versions "1-10"
```

### 4. Combinar Rangos y Versiones

```bash
python cleanup_layers.py --env develop --versions "1-5,8,10-15"
```

### 5. Modo Dry-Run (Simulación)

```bash
python cleanup_layers.py --env develop --versions "1-20" --dry-run
```

**Salida:**
```
🧹 Limpieza de Lambda Layer: EdutinLayerDevelop
======================================================================

🎯 Versiones a procesar: 20
   Rango: 1 - 20

======================================================================
🔍 ANALIZANDO VERSIONES

✅ Versión   1 - Segura para eliminar
✅ Versión   2 - Segura para eliminar
🔴 Versión   3 - EN USO por 2 función(es)
       - api-gateway-handler
       - data-processor
✅ Versión   4 - Segura para eliminar
...
🔒 Versión  18 - PROTEGIDA (versión reciente)
🔒 Versión  19 - PROTEGIDA (versión reciente)
🔒 Versión  20 - PROTEGIDA (versión reciente)

======================================================================
📊 RESUMEN

   Total especificadas:  20
   🟢 Seguras:           15
   🔴 En uso:            2
   🔒 Protegidas:        3

📋 Versiones a eliminar: [1, 2, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16]

⚠️  WARNING: 2 versiones están en uso y NO serán eliminadas

🔒 INFO: 3 versiones están protegidas y NO serán eliminadas

======================================================================
🧪 MODO DRY-RUN - No se eliminará nada
   Se eliminarían 15 versiones:
   - Versión 1
   - Versión 2
   - Versión 4
   ...
======================================================================
```

### 6. Eliminar con Confirmación

```bash
python cleanup_layers.py --env develop --versions "1-10"
```

**Interactivo:**
```
======================================================================
⚠️  ADVERTENCIA: Esta acción es IRREVERSIBLE

¿Eliminar 8 versiones? (escribe 'ELIMINAR' para confirmar): ELIMINAR

======================================================================
🗑️  ELIMINANDO VERSIONES

   Eliminando versión 1... ✅
   Eliminando versión 2... ✅
   Eliminando versión 4... ✅
   Eliminando versión 5... ✅
   Eliminando versión 6... ✅
   Eliminando versión 7... ✅
   Eliminando versión 8... ✅
   Eliminando versión 9... ✅

======================================================================
📊 RESULTADO FINAL

   ✅ Eliminadas:  8
   ❌ Fallidas:    0
   🔴 Omitidas:    1 (en uso)
   🔒 Protegidas:  3 (recientes)
======================================================================

🎉 ¡Limpieza completada!
```

### 7. Forzar Eliminación (Sin Confirmación)

```bash
# ⚠️ Usar con cuidado en producción
python cleanup_layers.py --env develop --versions "1-10" --force
```

### 8. Cambiar Versiones Protegidas

```bash
# Proteger las últimas 5 versiones en lugar de 3
python cleanup_layers.py --env develop --versions "1-20" --keep-last 5
```

### 9. Producción

```bash
# Listar versiones en producción
python cleanup_layers.py --env production --list

# Eliminar con dry-run primero
python cleanup_layers.py --env production --versions "1-10" --dry-run

# Si todo se ve bien, ejecutar
python cleanup_layers.py --env production --versions "1-10"
```

---

## 🔒 Opciones Avanzadas

### Protección de Versiones Recientes

Por defecto, las últimas 3 versiones están protegidas:

```bash
# Cambiar a 5 versiones protegidas
python cleanup_layers.py --env develop --versions "1-50" --keep-last 5
```

### Versiones en Uso

Las versiones usadas por funciones Lambda **nunca** se eliminan:

```
🔴 Versión   3 - EN USO por 2 función(es)
       - api-gateway-handler
       - data-processor
```

### Confirmación de Seguridad

Para eliminar, debes escribir exactamente `ELIMINAR`:

```bash
¿Eliminar 8 versiones? (escribe 'ELIMINAR' para confirmar): ELIMINAR
```

Cualquier otro input cancela la operación.

---

## 🛡️ Seguridad

### Verificaciones Automáticas

1. **Credenciales AWS**: Verifica que estén configuradas
2. **Existencia de Layer**: Valida que la layer exista
3. **Versiones válidas**: Filtra versiones que no existen
4. **Uso activo**: Verifica cada función Lambda
5. **Protección de recientes**: Protege últimas N versiones

### Flujo de Seguridad

```
1. Listar versiones
    ↓
2. Verificar uso en Lambdas
    ↓
3. Aplicar protecciones
    ↓
4. Mostrar resumen
    ↓
5. Pedir confirmación (si no --force)
    ↓
6. Eliminar solo versiones seguras
    ↓
7. Reportar resultados
```

### Mejores Prácticas

**✅ Recomendado:**
```bash
# 1. Siempre listar primero
python cleanup_layers.py --env develop --list

# 2. Usar dry-run
python cleanup_layers.py --env develop --versions "1-20" --dry-run

# 3. Ejecutar con confirmación
python cleanup_layers.py --env develop --versions "1-20"
```

**❌ No Recomendado:**
```bash
# Forzar sin revisar primero
python cleanup_layers.py --env production --versions "1-50" --force
```

---

## 🐛 Troubleshooting

### Error: No AWS credentials found

**Problema:**
```
❌ ERROR: No AWS credentials found. Please set your credentials with `aws configure`.
```

**Solución:**
```bash
aws configure
# Proporcionar:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region name
# - Default output format
```

### Error: Layer no encontrada

**Problema:**
```
❌ ERROR: Layer 'EdutinLayerDevelop' no encontrada
```

**Solución:**
- Verificar que el nombre de la layer sea correcto
- Verificar que tengas permisos para acceder a la layer
- Verificar la región AWS correcta

### Warning: Versiones no encontradas

**Problema:**
```
⚠️  WARNING: Versiones no encontradas (serán ignoradas): [11, 12, 13]
```

**Causa:** Las versiones especificadas no existen.

**Solución:** Usar `--list` para ver versiones disponibles.

### Error: Algunas versiones no pudieron ser eliminadas

**Problema:**
```
   Eliminando versión 5... ❌
⚠️  Algunas versiones no pudieron ser eliminadas
```

**Posibles causas:**
- Permisos IAM insuficientes
- Version en uso (no detectada)
- Límite de rate de AWS API

**Solución:**
```bash
# Verificar permisos
aws iam get-user

# Ver logs de error en la salida del script

# Reintentar después de unos minutos
```

### Versiones protegidas no esperadas

**Problema:** Más versiones protegidas de las esperadas.

**Solución:**
```bash
# Ajustar número de versiones a proteger
python cleanup_layers.py --env develop --versions "1-20" --keep-last 1
```

---

## 📊 Estrategias de Limpieza

### 1. Limpieza Regular (Mensual)

```bash
# Listar versiones
python cleanup_layers.py --env develop --list > versions_log.txt

# Eliminar versiones antiguas (más de 1 mes)
python cleanup_layers.py --env develop --versions "1-50" --dry-run

# Si todo OK, ejecutar
python cleanup_layers.py --env develop --versions "1-50"
```

### 2. Limpieza Agresiva

```bash
# Mantener solo las últimas 2 versiones
python cleanup_layers.py --env develop --versions "1-100" --keep-last 2
```

### 3. Limpieza Conservadora

```bash
# Mantener últimas 10 versiones
python cleanup_layers.py --env develop --versions "1-50" --keep-last 10
```

### 4. Limpieza por Fecha

```bash
# Primero listar y anotar versiones antes de fecha X
python cleanup_layers.py --env develop --list

# Luego eliminar manualmente las versiones antiguas
python cleanup_layers.py --env develop --versions "1,2,3,4,5"
```

---

## 🔗 Scripts Relacionados

- [deploy.py](../deploy.py) - Desplegar nuevas versiones de la Layer
- [messages.py](../python/messages.py) - Mensajes del sistema

---

## 📚 Referencias

- [AWS Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
- [Lambda API Reference](https://docs.aws.amazon.com/lambda/latest/dg/API_DeleteLayerVersion.html)
- [Boto3 Lambda Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html)

---

**[⬅️ Volver al README principal](../README.md)**


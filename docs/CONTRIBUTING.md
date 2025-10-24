# Guía de Contribución

Gracias por tu interés en contribuir a **AWS Edutin Layer**. Esta guía te ayudará a empezar.

## 📋 Tabla de Contenidos

- [Código de Conducta](#código-de-conducta)
- [Cómo Contribuir](#cómo-contribuir)
- [Configuración del Entorno](#configuración-del-entorno)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Estándares de Código](#estándares-de-código)
- [Testing](#testing)
- [Proceso de Pull Request](#proceso-de-pull-request)
- [Deployment](#deployment)
- [Versionado](#versionado)

## 🤝 Código de Conducta

Este proyecto adhiere al Contributor Covenant. Al participar, se espera que mantengas este código.

### Nuestros Compromisos

- 🤗 Ser respetuoso con diferentes puntos de vista
- 💬 Aceptar críticas constructivas
- 🎯 Enfocarse en lo mejor para la comunidad
- 🙏 Mostrar empatía hacia otros miembros

## 🚀 Cómo Contribuir

### Reportar Bugs

Si encuentras un bug, por favor crea un issue con:

```markdown
**Descripción del Bug**
Descripción clara y concisa del problema.

**Pasos para Reproducir**
1. Ir a '...'
2. Ejecutar '...'
3. Ver error

**Comportamiento Esperado**
Qué debería suceder.

**Comportamiento Actual**
Qué sucede actualmente.

**Entorno**
- OS: [ej. Ubuntu 20.04]
- Python: [ej. 3.11]
- Versión de Layer: [ej. v2.5.0]

**Logs Relevantes**
```python
# Incluir logs o traceback
```

**Contexto Adicional**
Cualquier otra información relevante.
```

### Sugerir Funcionalidades

Para sugerir nuevas funcionalidades:

```markdown
**¿La funcionalidad está relacionada con un problema?**
Descripción clara del problema. Ej: "Siempre me frustro cuando..."

**Describe la solución que te gustaría**
Descripción clara de qué quieres que suceda.

**Alternativas Consideradas**
Otras soluciones que consideraste.

**Contexto Adicional**
Capturas de pantalla, mockups, etc.
```

### Contribuir Código

1. **Fork** el repositorio
2. **Crea** una rama desde `develop`
3. **Implementa** tus cambios
4. **Escribe** tests
5. **Documenta** tu código
6. **Haz commit** con mensajes descriptivos
7. **Push** a tu fork
8. **Crea** un Pull Request

## 🛠️ Configuración del Entorno

### Prerequisitos

```bash
# Python 3.10, 3.11 o 3.12
python --version

# AWS CLI configurado
aws --version
aws configure list

# Git
git --version
```

### Instalación Local

```bash
# Clonar repositorio
git clone https://github.com/edutin/aws_edutin_layer.git
cd aws_edutin_layer

# Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Instalar dependencias de desarrollo
pip install -r requirements-dev.txt
```

### requirements-dev.txt

```txt
# Testing
pytest==7.4.3
pytest-cov==4.1.0
pytest-mock==3.12.0
moto==4.2.10  # Mock AWS services

# Linting
flake8==6.1.0
black==23.12.1
isort==5.13.2

# Type checking
mypy==1.7.1
boto3-stubs[dynamodb,lambda,events,opensearchservice]

# Documentation
sphinx==7.2.6
sphinx-rtd-theme==2.0.0
```

### Configuración de Pre-commit Hooks

```bash
# Instalar pre-commit
pip install pre-commit

# Instalar hooks
pre-commit install
```

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        args: ['--max-line-length=100', '--extend-ignore=E203,W503']

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [boto3-stubs]
```

## 📁 Estructura del Proyecto

```
aws_edutin_layer/
├── python/
│   └── python/                  # Código de la layer
│       ├── ddb_client/          # Cliente DynamoDB
│       │   ├── __init__.py
│       │   ├── config.py
│       │   ├── constants.py
│       │   ├── searches.py
│       │   ├── updates.py
│       │   ├── helpers.py
│       │   └── utils.py
│       │
│       ├── aws_client/          # Cliente AWS Services
│       │   ├── aws_lambda.py
│       │   └── open_search/
│       │
│       └── utils/               # Utilidades
│           └── logs.py
│
├── tests/                       # Tests unitarios
│   ├── test_ddb_client/
│   ├── test_aws_client/
│   └── test_utils/
│
├── docs/                        # Documentación
│   ├── DDB_CLIENT.md
│   ├── AWS_CLIENT.md
│   ├── UTILS.md
│   └── CONTRIBUTING.md
│
├── deploy.py                    # Script de deployment
├── messages.py                  # Mensajes de deployment
├── requirements.txt             # Dependencias producción
├── requirements-dev.txt         # Dependencias desarrollo
├── .pre-commit-config.yaml      # Pre-commit hooks
├── setup.py                     # Setup para desarrollo local
└── README.md                    # Documentación principal
```

## 📐 Estándares de Código

### PEP 8

Seguimos [PEP 8](https://peps.python.org/pep-0008/) con algunas excepciones:

- Longitud máxima de línea: **100 caracteres**
- Indentación: **4 espacios**

### Type Hints

**Siempre** usar type hints en funciones públicas:

```python
from typing import Dict, List, Optional, Union

def dynamo_search(
    table: str,
    params: Dict[str, any],
    limit: bool = False,
    **args
) -> Dict[str, any]:
    """
    Busca registros en DynamoDB.
    
    Args:
        table: Nombre de la tabla
        params: Parámetros de búsqueda
        limit: Habilitar paginación
        **args: Argumentos adicionales
        
    Returns:
        Diccionario con Items, Count_Items y LastEvaluatedKey
        
    Raises:
        ValueError: Si la tabla no existe
        KeyError: Si falta una clave requerida
    """
    ...
```

### Docstrings

Usar Google Style docstrings:

```python
def complex_function(arg1: str, arg2: int, arg3: Optional[bool] = None) -> Dict[str, any]:
    """Descripción corta de la función.
    
    Descripción más detallada que explica qué hace la función,
    casos de uso, y cualquier comportamiento especial.
    
    Args:
        arg1: Descripción del primer argumento
        arg2: Descripción del segundo argumento
        arg3: Descripción del tercer argumento opcional.
            Defaults to None.
    
    Returns:
        Descripción del valor de retorno con su estructura:
        {
            'key1': 'Descripción',
            'key2': 'Descripción'
        }
    
    Raises:
        ValueError: Si arg1 está vacío
        TypeError: Si arg2 no es numérico
    
    Example:
        >>> result = complex_function('test', 42, True)
        >>> print(result['key1'])
        'value'
    
    Note:
        Información adicional importante sobre el uso.
    
    Warning:
        Advertencias sobre comportamiento inesperado.
    """
    ...
```

### Naming Conventions

```python
# ✅ Correcto
class DynamoComparison:      # PascalCase para clases
    pass

def dynamo_search():         # snake_case para funciones
    pass

RESERVED_WORDS = [...]       # UPPER_CASE para constantes

user_id = "123"              # snake_case para variables

# ❌ Incorrecto
class dynamo_comparison:     # Debe ser PascalCase
    pass

def DynamoSearch():          # Debe ser snake_case
    pass

reservedWords = [...]        # Debe ser UPPER_CASE
```

### Imports

Orden de imports según PEP 8:

```python
# 1. Standard library
import logging
import json
from typing import Dict, List, Optional

# 2. Third party
import boto3
from boto3.dynamodb.conditions import Key, Attr

# 3. Local imports
from ddb_client.utils import describe_schema
from ddb_client.constants import RESERVED_WORDS

# isort se encarga de esto automáticamente
```

### Code Formatting

Usar `black` para formateo automático:

```bash
# Formatear todo el proyecto
black python/

# Verificar sin modificar
black python/ --check

# Formatear un archivo específico
black python/python/ddb_client/searches.py
```

### Linting

```bash
# Verificar código
flake8 python/

# Con configuración personalizada
flake8 python/ --max-line-length=100 --extend-ignore=E203,W503
```

### Type Checking

```bash
# Verificar tipos
mypy python/

# Con configuración
mypy python/ --ignore-missing-imports
```

## 🧪 Testing

### Estructura de Tests

```
tests/
├── conftest.py                  # Fixtures compartidos
├── test_ddb_client/
│   ├── test_searches.py
│   ├── test_updates.py
│   ├── test_helpers.py
│   └── test_utils.py
├── test_aws_client/
│   ├── test_lambda.py
│   └── test_opensearch.py
└── test_utils/
    └── test_logs.py
```

### Escribir Tests

```python
# tests/test_ddb_client/test_searches.py

import pytest
from moto import mock_dynamodb
import boto3
from ddb_client.searches import dynamo_search

@pytest.fixture
def dynamodb_table():
    """Fixture para crear tabla de prueba"""
    with mock_dynamodb():
        dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
        
        table = dynamodb.create_table(
            TableName='test_table',
            KeySchema=[
                {'AttributeName': 'user_id', 'KeyType': 'HASH'},
                {'AttributeName': 'curso_id', 'KeyType': 'RANGE'}
            ],
            AttributeDefinitions=[
                {'AttributeName': 'user_id', 'AttributeType': 'S'},
                {'AttributeName': 'curso_id', 'AttributeType': 'S'}
            ],
            BillingMode='PAY_PER_REQUEST'
        )
        
        # Insertar datos de prueba
        table.put_item(Item={
            'user_id': '123',
            'curso_id': '456',
            'estado': 1,
            'score': 95
        })
        
        yield table

def test_dynamo_search_basic(dynamodb_table):
    """Test búsqueda básica"""
    params = {
        'user_id': '123',
        'curso_id': '456'
    }
    
    result = dynamo_search('test_table', params)
    
    assert result['Count_Items'] == 1
    assert result['Items'][0]['user_id'] == '123'
    assert result['Items'][0]['score'] == 95

def test_dynamo_search_not_found(dynamodb_table):
    """Test cuando no se encuentran resultados"""
    params = {
        'user_id': '999'
    }
    
    result = dynamo_search('test_table', params)
    
    assert result['Count_Items'] == 0
    assert result['Items'] == []

def test_dynamo_search_with_filter(dynamodb_table):
    """Test búsqueda con filtro"""
    from ddb_client.helpers import DynamoComparison as dc
    
    params = {
        'user_id': '123',
        'estado': dc.Eq(1)
    }
    
    result = dynamo_search('test_table', params)
    
    assert result['Count_Items'] >= 1
```

### Ejecutar Tests

```bash
# Todos los tests
pytest

# Con coverage
pytest --cov=python/python --cov-report=html

# Solo un módulo
pytest tests/test_ddb_client/

# Solo un test específico
pytest tests/test_ddb_client/test_searches.py::test_dynamo_search_basic

# Con verbose
pytest -v

# Con logs
pytest -v -s
```

### Coverage Mínimo

Mantenemos un coverage mínimo de **80%**:

```bash
pytest --cov=python/python --cov-report=term --cov-fail-under=80
```

## 🔀 Proceso de Pull Request

### 1. Crear Rama

```bash
# Actualizar develop
git checkout develop
git pull origin develop

# Crear rama feature
git checkout -b feature/nombre-descriptivo

# O rama fix
git checkout -b fix/nombre-del-bug
```

### 2. Convenciones de Commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Tipos de commits
feat:     Nueva funcionalidad
fix:      Corrección de bug
docs:     Cambios en documentación
style:    Formateo, punto y coma faltantes, etc.
refactor: Refactorización de código
test:     Agregar o modificar tests
chore:    Tareas de mantenimiento
perf:     Mejora de performance
ci:       Cambios en CI/CD

# Ejemplos
git commit -m "feat: agregar soporte para operador OR en helpers"
git commit -m "fix: corregir paginación infinita en dynamo_search"
git commit -m "docs: actualizar README con ejemplos de OpenSearch"
git commit -m "test: agregar tests para batch_create_items"
```

### 3. Commit Guidelines

```bash
# ✅ Buenos commits
feat: agregar función de búsqueda full-text en OpenSearch
fix: resolver error de tipo en dynamo_increase
docs: actualizar guía de contribución
test: agregar tests unitarios para DynamoComparison

# ❌ Malos commits
update code
fix bug
changes
wip
```

### 4. Push y PR

```bash
# Push a tu fork
git push origin feature/nombre-descriptivo

# Crear PR en GitHub
# - Base: develop
# - Head: tu-usuario:feature/nombre-descriptivo
```

### 5. Template de PR

```markdown
## Descripción
Descripción clara de los cambios realizados.

## Tipo de Cambio
- [ ] Bug fix (cambio no-breaking que corrige un issue)
- [ ] Nueva funcionalidad (cambio no-breaking que agrega funcionalidad)
- [ ] Breaking change (cambio que causa que funcionalidad existente no funcione)
- [ ] Documentación

## ¿Cómo se ha probado?
Describe las pruebas realizadas.

- [ ] Tests unitarios
- [ ] Tests de integración
- [ ] Tests manuales

## Checklist
- [ ] Mi código sigue las guías de estilo del proyecto
- [ ] He realizado una auto-revisión de mi código
- [ ] He comentado mi código en áreas difíciles de entender
- [ ] He actualizado la documentación correspondiente
- [ ] Mis cambios no generan nuevos warnings
- [ ] He agregado tests que prueban mi cambio
- [ ] Tests nuevos y existentes pasan localmente
- [ ] Cambios dependientes han sido mergeados

## Screenshots (si aplica)
Capturas de pantalla relevantes.

## Notas Adicionales
Información adicional para los reviewers.
```

### 6. Code Review

Criterios de revisión:

- ✅ Código sigue estándares
- ✅ Tests incluidos y pasan
- ✅ Documentación actualizada
- ✅ Sin código comentado
- ✅ Sin console.logs o prints de debug
- ✅ Type hints correctos
- ✅ Docstrings completos

## 🚀 Deployment

### Proceso de Deployment

```bash
# 1. Asegurar rama correcta
git checkout develop  # Para development
git checkout master   # Para production

# 2. Deploy a desarrollo
python deploy.py --env develop

# 3. Deploy a producción (solo desde master)
git checkout master
python deploy.py --env production --message "Descripción del release"
```

### Validación Pre-Deployment

```bash
# Tests
pytest

# Linting
flake8 python/
black python/ --check
isort python/ --check

# Type checking
mypy python/
```

### CI/CD (GitHub Actions)

`.github/workflows/test.yml`:

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: Lint with flake8
      run: flake8 python/
    
    - name: Format check with black
      run: black python/ --check
    
    - name: Type check with mypy
      run: mypy python/ --ignore-missing-imports
    
    - name: Test with pytest
      run: pytest --cov=python/python --cov-report=xml
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## 📌 Versionado

Seguimos [Semantic Versioning](https://semver.org/):

```
MAJOR.MINOR.PATCH

MAJOR: Cambios incompatibles en API
MINOR: Nueva funcionalidad compatible
PATCH: Bug fixes compatibles
```

Ejemplos:
- `1.0.0` → `1.0.1`: Bug fix
- `1.0.1` → `1.1.0`: Nueva funcionalidad
- `1.1.0` → `2.0.0`: Breaking change

### Changelog

Mantener `CHANGELOG.md` actualizado:

```markdown
# Changelog

## [Unreleased]
### Added
- Nueva función para búsqueda avanzada

### Changed
- Mejorado rendimiento de batch operations

### Fixed
- Corregido error en paginación

## [1.2.0] - 2024-01-15
### Added
- Soporte para operador OR en helpers
- Nueva función op_bulk_create en OpenSearch

### Changed
- Actualizado boto3 a 1.34.0

### Fixed
- Corregido memory leak en batch operations
```

## 🤔 Preguntas Frecuentes

**P: ¿Puedo contribuir sin experiencia en AWS?**  
R: Sí, puedes empezar con documentación, tests, o pequeñas mejoras.

**P: ¿Cuánto tiempo toma el review de PR?**  
R: Típicamente 1-3 días hábiles.

**P: ¿Dónde pido ayuda?**  
R: Crea un issue o contacta en Slack #aws-layers.

**P: ¿Necesito firmar un CLA?**  
R: No por ahora, pero puede requerirse en el futuro.

## 📞 Contacto

- 💬 **Slack**: #aws-layers
- 📧 **Email**: dev@edutin.com
- 🐛 **Issues**: [GitHub Issues](link)

---

**¡Gracias por contribuir! 🎉**


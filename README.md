# DynamoDB Utility Library Documentation

This library provides simplified functions for interacting with AWS DynamoDB tables. The documented functions allow searching for records with specific conditions, retrieving batches of items using primary keys, inserting records, and updating or deleting data.

## Table of Contents

- **[Imports](#imports)**
- **[Functions](#functions)**
    - [dynamo_search](#dynamo_search)
    - [dynamo_counter](#dynamo_counter)
    - [batch_get_items](#batch_get_items)
- **[Helpers](#helpers)**
    - [DynamoComparison](#dynamocomparison)
- **[Logging](#logging)**
- **[Additional Considerations](#additional-considerations)**
- **[Limitations](#limitations)**


## Imports
```python
from ddb_client import (
    # Searches
    dynamo_search,
    batch_get_items,
    dynamo_counter,
    # Updates
    dynamo_update,
    dynamo_create
)
```

## Functions

### dynamo_search

The `dynamo_search` function allows searching for records in a DynamoDB table using specific key conditions and optional filters. *[Support helpers](#helpers)* for filterExpressions and Sort Keys

#### Function Definition

```python
def dynamo_search(table, params, limit=False, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table to query.
- **params** *(dict)*: Dictionary containing the search parameters, including primary key values and optional filters.
  - Keys in `params` can include specific `fields` and any attribute conditions for the sort key.
- **limit** *(bool, optional)*: If `True`, limits the number of pages retrieved according to the value configured in `config.pages` or passed through the `pages` parameter.
- **args** *(optional)*: Additional parameters such as:
  - **index** *(str)*: Name of the index defined in the DynamoDB schema for the search.
  - **paginate** *(bool, optional)*: Defines whether to paginate items. Default value in `config.py`.
  - **order** *(str, optional)*: Query order (`ASC` or `DESC`). Default: `ASC`.
  - **order_by** *(str, optional)*: Key by which the query is organized, typically a local secondary index (LSI).
  - **pages** *(int, optional)*: Number of items per page, default value in `config.py`.

#### Return

A dictionary with the structure:

```js
{
    "Items": [],           // List of retrieved items.
    "Count_Items": 0,      // Count of retrieved items.
    "LastEvaluatedKey": {} // (optional) Last evaluated key for pagination.
}
```

#### Usage Examples

```python
params = {
    'curso_id': '1234',
    'user_id': '425034'
}

result = dynamo_search('bas_certifications', params)
print(result)
```

```python
params = {
    'user_id': '425034',
    'fields': ['id', 'curso_id', 'estado'],  # Fields to be returned
}

result = dynamo_search('bas_certifications', params, limit=True, order_by='user_date')
print(result)
```

#### Details

`dynamo_search` performs queries using primary and sort keys, allowing ordering of results and applying filters. It supports pagination if `limit` is `True` and uses DynamoDB's `query` API for efficient retrieval.

### dynamo_counter

The `dynamo_counter` function counts the total number of items that meet certain conditions in a DynamoDB table. *[Support helpers](#helpers)* for filterExpressions

#### Function Definition

```python
def dynamo_counter(table, params):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table to query.
- **params** *(dict)*: Dictionary containing the search parameters.

#### Return

A dictionary with the structure:

```js
{
    "Count_Items": 0,  // Total number of items meeting the conditions.
    "Pages": 0         // Total number of pages, considering pagination settings.
}
```

#### Usage Example

```python
params = {
    'user_id': '425034'
}

result = dynamo_counter('bas_certifications', params)
print(result)
```

#### Details

`dynamo_counter` uses the `query` operation with the `'count'` type to efficiently count records that match the conditions specified in `params`.

### batch_get_items

The `batch_get_items` function retrieves multiple items in a single call to DynamoDB by specifying their primary keys.

#### Function Definition

```python
def batch_get_items(table, source, **keys_fields):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **source** *(list)*: List of dictionaries containing the primary key values of the items to retrieve.
- **keys_fields** *(optional)*: Additional options:
  - **xfields**: Fields to retrieve in the `ProjectionExpression`.
  - **logger_keys**: Log the keys used to retrieve items.
  - **logger_result**: Log the retrieved items.

#### Return
A list of items retrieved from the table.

#### Usage Example

```python
source = [
    {'id': 'value1', 'date': 'value2', 'user_id': 'value3'},
    {'id': 'value2', 'date': 'value2', 'user_id': 'value3'}
]

items = batch_get_items('bas_users', source, user_id='user_id')
print(items)
```

#### Details

`batch_get_items` uses DynamoDB's `batch_get_item` API to fetch items based on primary keys. The function handles batches of up to 100 items per call, managing unprocessed keys to ensure all items are retrieved.

---

## Helpers

### DynamoComparison

The `DynamoComparison` class allows creating queries with custom operators to facilitate building conditions in DynamoDB.

#### Class Definition

```python
class DynamoComparison:
    def __init__(self, operator, value=None, start=None, end=None, is_key=False):
        ...

    def build(self, field):
        ...

    # Equal to
    @staticmethod
    def Eq(value, is_key=False):
        ...

    # Not equal to
    @staticmethod
    def Ne(value, is_key=False):
        ...

    # Less than or equal to
    @staticmethod
    def Le(value, is_key=False):
        ...

    # Less than (Lt)
    @staticmethod
    def Lt(value, is_key=False):
        ...

    # Greater than or equal to (Ge)
    @staticmethod
    def Ge(value, is_key=False):
        ...

    # Greater than (Gt)
    @staticmethod
    def Gt(value, is_key=False):
        ...

    # Exists (NotNull)
    @staticmethod
    def NotNull(is_key=False):
        ...

    # Does not exist (Null)
    @staticmethod
    def Null(is_key=False):
        ...

    # Contains (Contains)
    @staticmethod
    def Contains(value, is_key=False):
        ...

    # Does not contain (NotContains)
    @staticmethod
    def NotContains(value, is_key=False):
        ...

    # Begins with (BeginsWith)
    @staticmethod
    def BeginsWith(value, is_key=False):
        ...

    # In a list (In)
    @staticmethod
    def In(values, is_key=False):
        ...

    # Between two values (Between)
    @staticmethod
    def Between(start, end, is_key=False):
        ...
```

#### Usage Example

```python
from ddb_client.helpers import DynamoComparison as dynamo_cmp
from ddb_client.search import dynamo_search

params = {
    'user_id': '425034',
    'course_class_source': dynamo_cmp.BeginsWith(12365, is_key=True)
}

result = dynamo_search('recents', params, limit=True)
print(result)
```
> [!NOTE]  
> The `is_key` parameter indicates whether it is a table key and should be constructed with the `Key` class instead of `Attr`.

---

## Logging

- In `batch_get_items`, use `logger_keys` and `logger_result` to log the keys and retrieved results.
- In `dynamo_search`, use `log_qparams` to log the constructed `qparams`.
- Errors during execution are logged with detailed information about the issue.

## Additional Considerations

- Keep in mind DynamoDB's pricing model, as read capacity units (RCU) and write capacity units (WCU) are consumed based on read consistency and the number of operations.
- To improve performance, ensure appropriate indexes are defined and used when needed.

## Limitations

- `batch_get_items` is limited to retrieving a maximum of 100 items per batch due to DynamoDB restrictions.
- `dynamo_search` supports searches based on primary and sort keys; more complex queries may require additional customization.

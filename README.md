# DynamoDB Utility Library Documentation

This library provides simplified functions for interacting with AWS DynamoDB tables. The documented functions allow searching for records with specific conditions, retrieving batches of items using primary keys, inserting records, updating or deleting data, and increasing specific field values.

## Table of Contents
- **[ddb_client](#ddb_client-dynamodb-utility-framework)**
    - *[Imports](#imports)*
    - [Searches Module](#-ddb_clientsearches-search-and-query-module)
        - [dynamo_search](#dynamo_search)
        - [dynamo_counter](#dynamo_counter)
        - [batch_get_items](#batch_get_items)
    - [Updates Module](#-ddb_clientupdates-update-and-modification-module)
        - [dynamo_create](#dynamo_create)
        - [batch_create_items](#batch_create_items)
        - [dynamo_update](#dynamo_update)
        - [dynamo_increase](#dynamo_increase)
        - [dynamo_delete](#dynamo_delete)
        - [batch_delete_items](#batch_delete_items)
    - **[Helpers](#helpers)**
        - [DynamoComparison](#dynamocomparison)
    - **[Additional Considerations](#additional-considerations)**
    - **[Limitations](#limitations)**


# ddb_client: DynamoDB Utility Framework

The `ddb_client` module is a framework designed to simplify interactions with AWS DynamoDB. It provides a set of high-level functions that abstract common database operations, making it easier to perform searches, batch operations, updates, and data manipulation without dealing with low-level AWS SDK complexities.

## Features
- **Search & Query Support**: Retrieve data with advanced filtering and sorting.
- **Batch Operations**: Efficiently insert and delete multiple records in a single request.
- **Incremental Updates**: Increase or decrease numerical values within DynamoDB attributes.
- **Schema Awareness**: Ensures operations align with predefined table schemas.
- **Error Handling & Logging**: Provides structured logs and error feedback for better debugging.

## Imports
```python
# Search Module
from ddb_client.searches import (
    dynamo_search,
    batch_get_items,
    dynamo_counter
)

# Update Module
from ddb_client.updates import (
    dynamo_update,
    dynamo_create,
    dynamo_increase,
    dynamo_delete,
    batch_create_items,
    batch_delete_items
)
```


# 📌 ddb_client.searches: Search and Query Module

The `ddb_client.searches` module is responsible for handling search and query operations in AWS DynamoDB. It provides efficient methods for retrieving records based on primary keys, applying filters, paginating results, and aggregating data. This module abstracts the complexity of DynamoDB queries, offering a user-friendly interface to perform structured searches with sorting and filtering capabilities.

## Features
- **Key-based queries**: Retrieve records using primary and sort keys.
- **Index support**: Perform searches on Global Secondary Indexes (GSI) and Local Secondary Indexes (LSI).
- **Filtering**: Apply filter expressions to refine search results.
- **Sorting**: Order results in ascending or descending order.
- **Pagination**: Retrieve data in batches with support for continuation tokens.
- **Aggregations**: Efficiently count matching records without retrieving full datasets.
- **Logging & Debugging**: Optionally log query parameters and schema details for troubleshooting.

## Usage
This module is particularly useful for applications that need to perform structured searches in DynamoDB while ensuring efficient data retrieval. It is ideal for:
- Fetching user-related records based on identifiers.
- Searching within large datasets with filtering conditions.
- Counting entries that meet specific criteria.
- Implementing paginated views in frontend applications.

By leveraging `ddb_client.searches`, developers can write cleaner, more maintainable code while optimizing database interactions.



# dynamo_search
The `dynamo_search` function allows searching for records in a DynamoDB table using specific key conditions and optional filters.
> 
> The dynamo_search function *[Support helpers](#helpers)* for Sort Keys and FilterExpressions
> Please note the minimum configuration in the *[config.py](#searches-module-cofiguration)* file to use this module

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
  - **order** *(str, optional)*: Query order (`ASC` or `DESC`). Default: `ASC`.
  - **order_by** *(str, optional)*: Key by which the query is organized, typically a local secondary index (LSI).
  - **pages** *(int, optional)*: Number of items per page, default value in `config.py`.
  - **log_errors** *(bool, opcional)*:Default: `True`. Display the details of errors that may occur when executing the query. 
  - **log_qparams** *(bool, optional)*: Displays the query that was built to perform the query. Default: `False`
  - **log_schema** *(bool, optional)*: Displays the schema of table which will be used to perform the query. Default: `False`


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

Common
```python
params = {
    'curso_id': '1234',
    'user_id': '425034'
}

result = dynamo_search('bas_certifications', params)
print(result)
```

Specific fields
```python
params = {
    'user_id': '425034',
    'fields': ['id', 'curso_id', 'estado'],  # Fields to be returned
}

result = dynamo_search('bas_certifications', params, limit=True, order_by='user_date')
print(result)
```

Filters or [Helpers](#helpers)
```python
from ddb_client.helpers import DynamoComparison as dynamo_cmp

params = {
    'user_id': '425034',
    'estado': dynamo_cmp.Ne(-1)  # All certifications active
}

result = dynamo_search('bas_certifications', params, limit=True)
print(result)
```

#### Details

`dynamo_search` performs queries using primary and sort keys, allowing ordering of results and applying filters. It supports pagination if `limit` is `True` and uses DynamoDB's `query` API for efficient retrieval.

# dynamo_counter

The `dynamo_counter` function counts the total number of items that meet certain conditions in a DynamoDB table.
> [!NOTE]
> The dynamo_counter function *[Support helpers](#helpers)* only for FilterExpressions

#### Function Definition

```python
def dynamo_counter(table, params, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table to query.
- **params** *(dict)*: Dictionary containing the search parameters.
- **args** *(optional)*: Additional parameters such as:
    - **log_qparams** *(bool, optional)*: Displays the query that was built to perform the query. Default: `False`
    - **log_schema** *(bool, optional)*: Displays the schema of table which will be used to perform the query. 
    - **log_errors** *(bool, opcional)*:Default: `True`. Display the details of errors that may occur when executing the query. 

#### Return

A dictionary with the structure:

```json
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

# batch_get_items

The `batch_get_items` function retrieves multiple items in a single call to DynamoDB by specifying their primary keys.

#### Function Definition

```python
def batch_get_items(table, source, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **source** *(list)*: List of dictionaries containing the primary key values of the items to retrieve.
- **args** *(optional)*: Additional options:
  - **xfields**: Fields to retrieve in the `ProjectionExpression`.
  - **logger_keys**: Log the keys used to retrieve items.
  - **logger_result**: Log the retrieved items.

#### Return
A list of items retrieved from the table.

#### Usage Example
With only Primary key
```python
source = [
    {'id': 'value1', 'date': 'value2', 'user_id': 'value3'},
    {'id': 'value2', 'date': 'value2', 'user_id': 'value3'}
]

items = batch_get_items('bas_users', source, user_id='user_id')
```
With Primary key and Sort key
```python
attempts = [
    {'id': 'value1', 'course_id': '4350', 'user_id': '425034'},
    {'id': 'value2', 'course_id': '4467', 'user_id': '568745'}
]
#                                                       Primary key        Sort key
items = batch_get_items('bas_certifications', attempts, user_id='user_id', curso_id='course_id')
```

Specifics fields
```python
attempts = [
    {'id': 'value1', 'course_id': '4350', 'user_id': '425034'},
    {'id': 'value2', 'course_id': '4467', 'user_id': '568745'}
]
#                                                       Primary key        Sort key
items = batch_get_items('bas_certifications', attempts, user_id='user_id', curso_id='course_id', fields=['user_id', 'user_name'])
```

#### Details

`batch_get_items` uses DynamoDB's `batch_get_item` API to fetch items based **ONLY on primary keys**. The function handles batches of up to 100 items per call, managing unprocessed keys to ensure all items are retrieved.

# 📌 ddb_client.updates: Update and Modification Module

The `ddb_client.updates` module is designed to simplify the process of modifying records in AWS DynamoDB. It provides efficient functions for updating attributes, inserting new records, batch operations, and safely incrementing numerical values. This module abstracts the complexity of DynamoDB update expressions, ensuring safe and optimized data modifications.

## Features
- **Single Record Updates**: Modify specific attributes within a record.
- **Batch Insert and Delete**: Efficiently process multiple items in a single operation.
- **Incremental Updates**: Increase or decrease attribute values safely with optional constraints.
- **Conditional Updates**: Ensure data integrity by enforcing conditions before modifications.
- **Schema Validation**: Automatically verifies operations against table schemas.
- **Logging & Debugging**: Optionally log update parameters and schema details for troubleshooting.

## Usage
This module is particularly useful for applications that require real-time data updates while ensuring efficient database interactions. Common use cases include:
- Updating user profiles and preferences.
- Incrementing counters for analytics or rate limiting.
- Performing batch inserts and deletions.
- Ensuring conditional updates to maintain data consistency.

By leveraging `ddb_client.updates`, developers can write clean, efficient, and maintainable code for modifying DynamoDB records while following best practices.


# dynamo_create

The `dynamo_create` function allows inserting a new item into a DynamoDB table.

#### Function Definition

```python
def dynamo_create(table, data, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **data** *(dict)*: Dictionary containing the item attributes and values to be inserted.
- **args** *(optional)*: Additional parameters such as:
    - **log_qparams** *(bool, optional)*: Default: `True`. Displays the query that was built to perform the query.
    - **log_errors** *(bool, opcional)*: Default: `True`. Display the details of errors that may occur when executing the query

#### Return
A dictionary with the structure:

```json
{
    "data": {},          // The inserted item.
    "status": true | false // Status of the operation.
}
```

> [!NOTE]
> The dynamo_create function added  Automatically add `create_at (utc iso format)`, `id (uuid)`, and `timestamp (number)` fields to records, only if they are not submitted. If the data source contains these attributes, ignore the default ones.

#### Usage Example

```python
data = {
    'name': 'New Item',
    'description': 'A new item to add to the table'
}

result = dynamo_create('bas_portfolio', data)
print(result)
```

# batch_create_items

The `batch_create_items` function allows inserting multiple items into a DynamoDB table efficiently using batch operations.

## Function Definition

```python
def batch_create_items(table, source, **args):
    ...
```

## Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **source** *(list)*: List of dictionaries containing the items to be inserted.
- **args** *(optional)*: Additional options:
  - **log_keys** *(bool, optional)*: If `True`, logs the keys of the items to be inserted.
  - **log_result** *(bool, optional)*: If `True`, logs the response from DynamoDB.

## Return

```json
{
    "status": true | false,  // Status of the operation.
    "row_count": 0 | N       // Number of rows inserted.
}
```

## Usage Example

```python
products = [
    {"product_id": "123", "name": "Item 1", "category": "Books"},
    {"product_id": "124", "name": "Item 2", "category": "Electronics"},
    {"product_id": "125", "name": "Item 3", "category": "Clothing"}
]

result = batch_create_items("product_catalog", items, id='product_id')
print(result)
```

### Details

- The function extracts primary and sort keys based on the schema of the table.
- It ensures that items with duplicate keys are not inserted.
- Uses DynamoDB's `batch_write_item` API, which allows inserting up to **25 items per batch**.
- Handles **UnprocessedItems**, retrying them automatically until all items are successfully inserted.
- Returns a count of inserted items and an operation status (`True` for success, `False` if no items were inserted).


# dynamo_update

The `dynamo_update` function allows updating attributes of an existing item in a DynamoDB table.

#### Function Definition

```python
def dynamo_update(table, keys, data, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **keys** *(dict)*: Dictionary specifying the primary key(s) of the item to update.
- **data** *(dict)*: Dictionary containing the attributes to be updated and their new values.
- **args** *(optional)*: Additional conditions that must be met for the update to proceed.
    - **log_qparams**: *(optional)*: Default `False`
    - **log_schema**: *(optional)*: Default `False`
    - **log_keys**: *(optional)*: Default `False`
    - **log_criticals**: *(optional)*: Default `True`
    - **log_errors** *(bool, opcional)*:Default: `True`. Display the details of errors that may occur when executing the query. 

#### Return
A dictionary with the structure:

```json
{
    "status": true | false,  // Status of the operation.
    "row_count": 0 | 1       // Number of rows affected by the operation.
}
```

#### Usage Example

```python
params = { 'id': 'item1' }
data = { 'description': 'Updated description' }

result = dynamo_update('bas_portfolio', params, data)
```

# dynamo_increase

The `dynamo_increase` function allows incrementing numerical values of specific attributes of an item in a DynamoDB table.

## Function Definition

```python
def dynamo_increase(table, params, data, **args):
    ...
```

## Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **params** *(dict)*: Dictionary specifying the primary key(s) of the item to update.
    - Supports optional **min** and **max** constraints for the attribute values.
- **data** *(dict)*: Dictionary containing the attributes to be incremented and the increment values.
- **args** *(optional)*: Additional parameters such as:
    - **log_schema** *(bool, optional)*: Default: `False`. Displays the schema of table which will be used to perform the query. 
    - **log_qparams** *(bool, optional)*:Default: `False`. Displays the query that was built to perform the query.
    - **log_errors** *(bool, opcional)*:Default: `True`. Display the details of errors that may occur when executing the query. 

## Return
A dictionary with the structure:

```json
{
    "status": true | false,  // Status of the operation.
    "row_count": 0 | 1       // Number of rows affected by the operation.
}
```

## Usage Example

```python
params = { "id": "item1" }
data = { "score": 2 }  # Increase the current value by 2

result = dynamo_increase("bas_certifications_statistics", params, data)
print(result)
```

```python
params = { 
    "id": "item1", 
    'score_min': 0 # The minimum value it will reach will be 0
} 
data = { "score": 1 }  # Increase the current value by 2

result = dynamo_increase("bas_certifications_statistics", params, data)
print(result)
```

## Details

- The function constructs an update expression to increment attributes.
- Ensures that the attribute exists before applying the increment.
- Can enforce **min** and **max** constraints to prevent values from exceeding defined limits.
- Uses DynamoDB's `update_item` API for efficient modification.
- Returns a status indicating success and the number of affected rows.



# dynamo_delete

The `dynamo_delete` function allows deleting an existing item from a DynamoDB table.

#### Function Definition

```python
def dynamo_delete(table, params, **args):
    ...
```

#### Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **params** *(dict)*: Dictionary specifying the primary key(s) of the item to delete.
- **args** *(optional)*: Additional parameters such as:
    - **return_values** *(bool, optional)*: Default `True`. Returns the old values ​​before it was deleted
    - **log_schema** *(bool, optional)*: Default: `False`. Displays the schema of table which will be used to perform the query. 
    - **log_qparams** *(bool, optional)*:Default: `False`. Displays the query that was built to perform the query.
    - **log_errors** *(bool, opcional)*:Default: `True`. Display the details of errors that may occur when executing the query. 

#### Return
A dictionary with the structure:

```json
{
    "status": true | false,  // Status of the operation.
    "row_count": 0 | 1       // Number of rows affected by the operation.
}
```

#### Usage Example

```python
params = { 'id': 'item1' }

result = dynamo_delete('bas_portfolio', params)
```

---

# batch_delete_items

The `batch_delete_items` function allows deleting multiple items from a DynamoDB table efficiently using batch operations.

## Function Definition

```python
def batch_delete_items(table, source, **args):
    ...
```

## Parameters

- **table** *(str)*: Name of the DynamoDB table.
- **source** *(list)*: List of dictionaries containing the primary key values of the items to delete.
- **args** *(optional)*: Additional options:
  - **log_keys** *(bool, optional)*: If `True`, logs the keys of the items to be deleted.
  - **log_result** *(bool, optional)*: If `True`, logs the response from DynamoDB.

## Return
A dictionary with the structure:

```json
{
    "status": true | false,  // Status of the operation.
    "row_count": 0 | N       // Number of rows deleted.
}
```

## Usage Example

```python
items_to_delete = [
    {"id": "123"},
    {"id": "124"},
    {"id": "125"}
]

result = batch_delete_items("product_catalog", items_to_delete, id='id')
print(result)
```

## Details

- The function extracts primary and sort keys based on the schema of the table.
- It ensures that duplicate keys are not processed multiple times.
- Uses DynamoDB's `batch_write_item` API, which allows deleting up to **25 items per batch**.
- Handles **UnprocessedItems**, retrying them automatically until all items are successfully deleted.
- Returns a count of deleted items and an operation status (`True` for success, `False` if no items were deleted).


# Helpers

## DynamoComparison

The `DynamoComparison` class allows creating queries with custom operators to facilitate building conditions in DynamoDB.

### Class Definition

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
from ddb_client import dynamo_search

params = {
    'user_id': '425034',
    'course_class_source': dynamo_cmp.BeginsWith(12365, is_key=True)
}

result = dynamo_search('recents', params, limit=True)
print(result)
```
> [!NOTE]  
> The `is_key` parameter indicates whether it is a table key and should be constructed with the `Key` class instead of `Attr`.


## Configurations
### Searches module cofiguration
```python
# config.py

paginate = True # Required
page = False    # Required


# Value allowed to assign (Gets all fields from the table you want to query)
fields = {} 

# Else you can specify the fields
fields = {
    'table1': [
        'field1',
        'field2',
        'field3',
        'field4',
    ],

    'table2': [
        'field1',
        'field2',
        'field3',
        'field4',
    ]
}
```

### Searches module cofiguration
```python
# config.py


# Value allowed to assign
allowed_fields = {} 

# Else you can specify the fields
allowed_fields = {
    'table2': [
        'field1',
        'field1.sub_field1',
        'field1.sub_field2',
        'field2',
        'field3',
    ]
}



defaults = {
    'data': {
        'table1': {
            'field1': 'value1'
            'field2': 'value2'
            'field3': 'value3'
            'field4': 'value4'
        },

        'table2': { } # REQUIRED! Is a valid value when dont have default values
    }
}
```



## Additional Considerations

- Keep in mind DynamoDB's pricing model, as read capacity units (RCU) and write capacity units (WCU) are consumed based on read consistency and the number of operations.
- To improve performance, ensure appropriate indexes are defined and used when needed.

## Limitations

- `batch operations` is limited to retrieving a maximum of 100 for query or 25 for update items per batch due to DynamoDB restrictions.
- `dynamo_search` supports searches based on primary and sort keys; more complex queries may require additional customization.
- `dynamo_create`, `dynamo_update`, `dynamo_increase`, and `dynamo_delete` functions rely on proper schema definition and allowed fields configurations, which must be set up appropriately to avoid runtime issues.


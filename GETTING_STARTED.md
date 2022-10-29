# Getting Started

## Auth - Connect to GDN
The first step to start using GDN is establishing a connection to [gdn.paas.macrometa.io](https://gdn.paas.macrometa.io/). When this code executes, it initializes the server connection to your nearest region. You can access your Macrometa GDN account using several methods, such as [API keys](https://macrometa.com/docs/account-management/api-keys/), [User authentication](https://macrometa.com/docs/account-management/auth/user-auth), [Token-based authentication (JWT)](https://macrometa.com/docs/account-management/auth/jwts). For this example the method used is API key.

```python
from c8 import C8Client
client = C8Client(protocol='https', host='gdn.paas.macrometa.io', port=443, apikey=<your api key>)
```
## Create a collection
A [Collection](https://macrometa.com/docs/collections/) consists of documents, and they can be local(only stores data in one region) or global(replicates data and maintains consistency across regions). Macrometa offers several types of collections such as [Document Store](https://macrometa.com/docs/collections/documents/), [Key-Value Store](https://macrometa.com/docs/collections/keyvalue/), [Dynamo Table](https://macrometa.com/docs/collections/dynamo/create-dynamo-table) and [Graph-Edge collection](https://macrometa.com/docs/collections/graphs/). For this example we are going to create a [Document Store](https://macrometa.com/docs/collections/documents/).

```python
client.create_collection(name='employees', sync=False, edge=False, local_collection=False, stream=False)
```
# CRUD Operations for collections
## Insert Document
Documents are stored in collections. Macrometa GDN is schemaless, so you do not need to define valid document attributes when creating your collection. Documents with completely different structures can be stored in the same collection.

**Inserting docs in 'employees' collection:**
```python
docs = [
    {'_key': 'Han', 'firstname': 'Han', 'lastname': 'Solo', 'email': 'han.solo@macrfabricometa.io', 'age': 35,
     'role': 'Manager'},
    {'_key': 'Bruce', 'firstname': 'Bruce', 'lastname': 'Wayne', 'email': 'bruce.wayne@mfabricacrometa.io', 'age': 40,
     'role': 'Developer', 'phone': '1-999-888-9999'},
    {'_key': 'Jon', 'firstname': 'Jon', 'lastname': 'Doe', 'email': 'jon.doe@macrfabricometa.io', 'age': 25,
     'role': 'Manager'},
    {'_key': 'Zoe', 'firstname': 'Zoe', 'lastname': 'Hazim', 'email': 'zoe.hazim@macrfabricometa.io', 'age': 20,
     'role': 'Director'},
    {'_key': 'Emma', 'firstname': 'Emma', 'lastname': 'Watson', 'email': 'emma.watson@macrfabricometa.io', 'age': 25,
     'role': 'Director'}
]
client.insert_document(collection_name='employees', document=docs)
```

## Read Document
Macrometa is robust. It uses [C8QL](https://macrometa.com/docs/queryworkers/c8ql/) to create, retrieve, modify and delete data stored in the globally distributed platform. This example demonstrates how to retrieve a particular document.

**Retreive an specific document with it's _key value:**
```python
client.get_document(collection='employees', document={'_key': 'Han'})
```

## Update Document
The [update](https://macrometa.com/docs/queryworkers/c8ql/got-tutorial/c8ql-crud#update-documents) operation can partially update a document in a collection. The document must contain the attributes and values to be updated and the _key attribute to identify the document to be updated. For this example the email for *Han* will be updated.

```python
client.update_document(collection_name='employees', document={'_key': 'Han', 'email': 'han@updated_macrometa.io'})
```

## Delete Document
The [delete](https://macrometa.com/docs/queryworkers/c8ql/got-tutorial/c8ql-crud#delete-documents) operation is used to remove a document from a collection. For this example *Han* will be removed from the **employees** collection.
```python
client.delete_document(collection_name='employees', document={'_key': 'Han'})
```

## Working with C8QL
Working with data can be complex. CRUD operations usually need more logic or conditions to reach the desired results. At Macrometa the [C8 query language (C8QL)](https://macrometa.com/docs/queryworkers/c8ql/) can be used to create, retrieve, modify and delete data that are stored in the Macrometa geo-distributed fast data platform.
**Check out the [operators](https://macrometa.com/docs/queryworkers/c8ql/) and [examples](https://macrometa.com/docs/queryworkers/c8ql/examples/) in Macrometa**

Let's [FILTER](https://macrometa.com/docs/queryworkers/c8ql/operations/filter) results by **role** and **age** and return a custom object using [ C8QL](https://macrometa.com/docs/queryworkers/c8ql/):
```python
query = "FOR doc IN employees FILTER doc.role == 'Manager' FILTER doc.age > 30 RETURN {'Name':doc.firstname,'Last Name':doc.lastname,'Email':doc.email}"
cursor = client.execute_query(query=query)
docs = [document for document in cursor]
```

It might not always be necessary to return all documents that a FOR loop would normally return. In those cases, you can limit the number of documents with a [LIMIT()](https://macrometa.com/docs/queryworkers/c8ql/operations/limit) operation. For this case, it will be used along with the [SORT()](https://macrometa.com/docs/queryworkers/c8ql/operations/sort) operation.
```python
query = "FOR doc IN employees SORT doc.age LIMIT 2 RETURN doc"
cursor = client.execute_query(query=query)
docs = [document for document in cursor]
```
[Grouping](https://macrometa.com/docs/queryworkers/c8ql/examples/grouping) results is a common operation when retrieving data. To group results by arbitrary criteria, C8QL provides the [COLLECT](https://macrometa.com/docs/queryworkers/c8ql/operations/collect) keyword. [COLLECT](https://macrometa.com/docs/queryworkers/c8ql/operations/collect) will perform a grouping, but no aggregation. In the following example let's group by **age** and return only the **first name** of the employee using the  expansion operator [*].

```python
query = "FOR doc IN employees COLLECT age = doc.age INTO employeesByAge RETURN {age, employee: employeesByAge[*].doc.firstname}"
cursor = client.execute_query(query=query)
docs = [document for document in cursor]
```

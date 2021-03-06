## Databases

Cloudant databases contain JSON objects.
These JSON objects are called [documents](#documents).
All documents must be contained in a database.

You interact with Cloudant databases and documents using API commands, as this overview explains:<br/>
<iframe width="280" height="158" src="https://www.youtube.com/embed/47qQMaYJVUU?rel=0" frameborder="0" allowfullscreen></iframe>

### Create

> Create a database

```http
PUT /$DATABASE HTTP/1.1
HOST: $ACCOUNT.cloudant.com
```

```shell
curl https://$USERNAME:$PASSWORD@$ACCOUNT.cloudant.com/$DATABASE -X PUT
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.db.create($DATABASE, function (err, body, headers) {
  if (!err) {
    console.log('database created!');
  }
});
```

To create a database, make a PUT request to `https://$USERNAME.cloudant.com/$DATABASE`.

The database name must start with a lowercase letter and contain only the following characters:

 - Lowercase characters (a-z)
 - Digits (0-9)
 - Any of the characters _, $, (, ), +, -, and /
 
#### Query Parameters

> Create a database with non-default values for `n` and `q`

```shell
curl -X PUT 'http://$USERNAME:$PASSWORD@$USERNAME.cloudant.com/$DATABASE?n=2&q=32'
```

```http
PUT /$DATABASE?n=2&q=32 HTTP/1.1
HOST: $ACCOUNT.cloudant.com
```

There are two configuration parameters that control the sharding topology of a database. The defaults are specified in the server configuration and may be overridden at database creation time on dedicated database clusters. N specifies the number of replicas of each document, while Q fixes the number of partitions of the database. On multi-tenant clusters, the default can not be overwritten.

Parameter | Description | Default
----------|-------------|----------
n         | number of replicas of each document | 3
q         | number of partitions of the database | 4

#### Response

> Response for successful creation:

```
HTTP/1.1 201 Created

{
  "ok": true
}
```

If creation succeeds, you get a 201 or 202 response. In case of an error, the HTTP status code tells you what went wrong.

Code | Description
-----|--------------
201 |	Database created successfully
202 |	The database has been successfully created on some nodes, but the number of nodes is less than the write quorum.
403 |	Invalid database name.
412 |	Database aleady exists.

### Read

> Create a database

```http
GET /$DATABASE HTTP/1.1
```

```shell
curl https://$USERNAME.cloudant.com/$DATABASE \
     -u $USERNAME
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.db.get($DATABASE, function (err, body, headers) {
  if (!err) {
    console.log(body);
  }
});
```

Making a GET request against `https://$USERNAME.cloudant.com/$DATABASE` returns details about the database,
such as how many documents it contains.


###### h6

> Example response:

```json
{
  "update_seq": "0-g1AAAADneJzLYWBgYMlgTmFQSElKzi9KdUhJMtbLTS3KLElMT9VLzskvTUnMK9HLSy3JAapkSmRIsv___39WIgOqHkM8epIcgGRSPTZt-KzKYwGSDA1ACqhzP0k2QrQegGgF2ZoFAGdBTTo",
  "db_name": "db",
  "purge_seq": 0,
  "other": {
    "data_size": 0
  },
  "doc_del_count": 0,
  "doc_count": 0,
  "disk_size": 316,
  "disk_format_version": 5,
  "compact_running": false,
  "instance_start_time": "0"
}
```

The elements of the returned structure are shown in the table below:

Field |	Description
------|------------
compact_running |	Set to true if the database compaction routine is operating on this database.
db_name |	The name of the database.
disk_format_version |	The version of the physical format used for the data when it is stored on disk.
disk_size |	Size in bytes of the data as stored on the disk. Views indexes are not included in the calculation.
doc_count |	A count of the documents in the specified database.
doc_del_count |	Number of deleted documents
instance_start_time |	Always 0.
purge_seq |	The number of purge operations on the database.
update_seq |	An opaque string describing the state of the database. It should not be relied on for counting the number of updates.
other |	Json object containing a data_size field.

### Get Databases

> Get all databases

```http
GET /_all_dbs HTTP/1.1
```

```shell
curl https://$USERNAME.cloudant.com/_all_dbs \
     -u $USERNAME
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.db.list(function (err, body, headers) {
  if (!err) {
    console.log(body);
  }
});
```

To list all the databases in an account,
make a GET request against `https://$USERNAME.cloudant.com/_all_dbs`.

###### h6

> Example response:

```json
[
   "_users",
   "contacts",
   "docs",
   "invoices",
   "locations"
]
```

The response is an array with all database names.

### Get Documents

> Getting all documents in a database

```http
GET /_all_docs HTTP/1.1
```

```shell
curl https://%USERNAME:$PASSWORD@$USERNAME.cloudant.com/$DATABASE/_all_docs
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");
var db = account.use($DATABASE);

db.list(function (err, body, headers) {
  if (!err) {
    console.log(body);
  }
});
```

To list all the documents in a database, make a GET request against `https://$USERNAME.cloudant.com/$DATABASE/_all_docs`.

The method accepts these query arguments:

Argument | Description | Optional | Type | Default
---------|-------------|----------|------|--------
`descending` | Return the documents in descending by key order | yes | boolean | false
`endkey` | Stop returning records when the specified key is reached | yes | string |  
`include_docs` | Include the full content of the documents in the return | yes | boolean | false
`inclusive_end` | Include rows whose key equals the endkey | yes | boolean | true
`key` | Return only documents that match the specified key | yes | string |  
`limit` | Limit the number of the returned documents to the specified number | yes | numeric | 
`skip` | Skip this number of records before starting to return the results | yes | numeric | 0
`startkey` | Return records starting with the specified key | yes | string |

###### h6

> Example response:

```json
{
  "total_rows": 3,
  "offset": 0,
  "rows": [{
    "id": "5a049246-179f-42ad-87ac-8f080426c17c",
    "key": "5a049246-179f-42ad-87ac-8f080426c17c",
    "value": {
      "rev": "2-9d5401898196997853b5ac4163857a29"
    }
  }, {
    "id": "96f898f0-f6ff-4a9b-aac4-503992f31b01",
    "key": "96f898f0-f6ff-4a9b-aac4-503992f31b01",
    "value": {
      "rev": "2-ff7b85665c4c297838963c80ecf481a3"
    }
  }, {
    "id": "d1f61e66-7708-4da6-aa05-7cbc33b44b7e",
    "key": "d1f61e66-7708-4da6-aa05-7cbc33b44b7e",
    "value": {
      "rev": "2-cbdef49ef3ddc127eff86350844a6108"
    }
  }]
}
```

The response is a JSON object containing all documents in the database matching the parameters. The following table describes the meaning of the individual fields:

Field |	Description |	Type
------|-------------|-------
offset |	Offset where the document list started |	numeric
rows |	Array of document objects |	array
total_rows |	Number of documents in the database/view matching the parameters of the query |	numeric
update_seq |	Current update sequence for the database |	string

### Get Changes

> Example request to get list of changes made to documents in a database:

```http
GET /$DATABASE/_changes HTTP/1.1
```

```shell
curl https://$USERNAME.cloudant.com/$DATABASE/_changes \
     -u $USERNAME
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.db.changes($DATABASE, function (err, body, headers) {
  if (!err) {
    console.log(body);
  }
});
```

> Example response:

```
{
  "results": [{
    "seq": "1-g1AAAAI9eJyV0EsKwjAUBdD4Ad2FdQMlMW3TjOxONF9KqS1oHDjSnehOdCe6k5oQsNZBqZP3HiEcLrcEAMzziQSB5KLeq0zyJDTqYE4QJqEo66NklQkrZUr7c8wAXzRNU-T22tmHGVMUapR2Bdwj8MBOvu4gscQyUtghyw-CYJ-SOWXTUSJMkKQ_UWgfsnXIuYOkhCCN6PBGqqmd4GKXda4OGvk0VCcCweHFeOjmoXubiEREIyb-KMdLDy89W4nTVGkqhhfkoZeHvkrimMJYrYo31bKsIg",
    "id": "foo",
    "changes": [{
      "rev": "1-967a00dff5e02add41819138abb3284d"
    }]
  }],
  "last_seq": "1-g1AAAAI9eJyV0EsKwjAUBdD4Ad2FdQMlMW3TjOxONF9KqS1oHDjSnehOdCe6k5oQsNZBqZP3HiEcLrcEAMzziQSB5KLeq0zyJDTqYE4QJqEo66NklQkrZUr7c8wAXzRNU-T22tmHGVMUapR2Bdwj8MBOvu4gscQyUtghyw-CYJ-SOWXTUSJMkKQ_UWgfsnXIuYOkhCCN6PBGqqmd4GKXda4OGvk0VCcCweHFeOjmoXubiEREIyb-KMdLDy89W4nTVGkqhhfkoZeHvkrimMJYrYo31bKsIg",
  "pending": 0
}
```

> Example response, continuous changes feed:

```
{
  "seq": "1-g1AAAAI7eJyN0EsOgjAQBuD6SPQWcgLSIm1xJTdRph1CCEKiuHClN9Gb6E30JlisCXaDbGYmk8mXyV8QQubZRBNPg6r2GGsI_BoP9YlS4auiOuqkrP0S68JcjhMCi6Zp8sxMO7OYISgUK3AF1iOAZyqsv8jog4Q6YIxyF4n6kLhFNs4nIQ-kUtJFwj5k2yJnB0lxSbkIhgdSTk0lF9OMc-0goCpikg7PxUI3C907KMKUM9AuJP9CDws9O0ghAtc4PB8LvSz0k5HgKTCU-RtU1qyw",
  "id": "2documentation22d01513-c30f-417b-8c27-56b3c0de12ac",
  "changes": [{
    "rev": "1-967a00dff5e02add41819138abb3284d"
  }]
}
{
  "seq": "2-g1AAAAI7eJyN0E0OgjAQBeD6k-gt5ASkRdriSm6iTDuEEIREceFKb6I30ZvoTbBYE-wG2cxMmubLyysIIfNsoomnQVV7jDUEfo2H-kSp8FVRHXVS1n6JdWF-jhMCi6Zp8sxcO_MwQ1AoVuAKrEcAz0xYf5HRBwl1wBjlLhL1IXGLbJwkIQ-kUtJFwj5k2yJnJ0mKS8pFMLyQcmomuZhlnGuXBqiKmKTDe7HQzUL3Doow5Qy0C8m_0MNCzw5SiMA1Du_HQi8L_RQteAoMZf4GVgissQ",
  "id": "1documentation22d01513-c30f-417b-8c27-56b3c0de12ac",
  "changes": [{
    "rev": "1-967a00dff5e02add41819138abb3284d"
  }]
}
{
  "seq": "3-g1AAAAI7eJyN0EsOgjAQBuD6SPQWcgLSIqW4kpso0w4hBCFRXLjSm-hN9CZ6EyyUBLtBNjOTyeTL5M8JIct0poijQJZHjBR4boWn6kJp4Mq8PKu4qNwCq1xfTmMCq7qus1RPB71YIEgMNmALbEAAR1fYdsikRXzlMUa5jYRDSNQgO-sTn3tCSmEj_hCyb5Brh0xbJME15YE3PpBiriu56aade_8NUBkyQcfnYqCHgZ49FGLCGSgbEn-hl4HePSQRgSscn4-BPgb6CTrgCTAU2RdXOqyy",
  "id": "1documentation22d01513-c30f-417b-8c27-56b3c0de12ac",
  "changes": [{
    "rev": "2-eec205a9d413992850a6e32678485900"
  }],
  "deleted": true
}
{
  "seq": "4-g1AAAAI7eJyN0EEOgjAQBdAGTfQWcgLSIm1xJTdRph1CCEKiuHClN9Gb6E30JlisCXaDbGYmTfPy80tCyDyfaOJrUPUeEw1h0OChOVEqAlXWR51WTVBhU5qfXkpg0bZtkZtrZx5mCArFClyBDQjgmwnrL-J9kEiHjFHuIvEQknTIxkkS8VAqJV0kGkK2HXJ2kmS4pFyE4wuppmaSi1nGufZpgKqYSTq-FwvdLHTvoRgzzkC7kPwLPSz07CGFCFzj-H4s9LLQT9GCZ8BQFm9Y9qyz",
  "id": "2documentation22d01513-c30f-417b-8c27-56b3c0de12ac",
  "changes": [{
    "rev": "2-eec205a9d413992850a6e32678485900"
  }],
  "deleted": true
}
```

Making a GET request against `https://$USERNAME.cloudant.com/$DATABASE/_changes` returns a list of changes made to documents in the database, including insertions, updates, and deletions. This log [might not be in chronological order](http://en.wikipedia.org/wiki/Clock_synchronization#Problems).

`_changes` accepts these query arguments:

Argument | Description | Supported Values | Default 
---------|-------------|------------------|---------
`descending` | Return the changes in sequential order | boolean | false | 
`doc_ids` | List of documents IDs to use to filter updates | array of strings with valid document IDs | all documents
`feed` | Type of feed | `"continuous"`, `"longpoll"`, `"normal"` | `"normal"`
`filter` | Name of filter function from a design document to get updates | string | no filter
`heartbeat` | Time in milliseconds after which an empty line is sent during longpoll or continuous if there have been no changes | any positive number | 60000 | 
`include_docs` | Include the document with the result | boolean | false |
`limit` | Maximum number of rows to return | any non-negative number | none |  
`since` | Start the results from changes after the specified sequence number. If since is 0 (the default), the request will return all changes. | string | 0 | 
`style` | Specifies how many revisions are returned in the changes array. The default, `main_only`, only returns the current "winning" revision; `all_docs` returns all leaf revisions, including conflicts and deleted former conflicts. | `main_only`, `all_docs` | `main_only` | 
`timeout` | Number of milliseconds to wait for data before terminating the response. If heartbeat supersedes timeout if both are supplied. | any positive number | |

All arguments are optional.

The `feed` argument changes how Cloudant sends the response. By default, changes feed in entirety and the connection closes.

If you set `feed=longpoll`, requests to the server remain open until changes are reported. This can help monitor changes specifically instead of continuously.

If you set `feed=continuous`, new changes send without closing the connection. In this mode the format of changes accomodates the continuous nature while ensuring validity of the JSON output.

The `filter` parameter designates a pre-defined [function to filter](#filter-functions) the changes feed.

### Delete

> Example request to delete a Cloudant database:

```http
DELETE /$DATABASE HTTP/1.1
Host: $USERNAME.cloudant.com
```

```shell
curl https://$USERNAME.cloudant.com/$DATABASE \
     -X DELETE \
     -u $USERNAME
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.db.destroy($DATABASE, function (err, body, headers) {
  if (!err) {
    console.log(body);
  }
});
```

To delete a databases and its contents, make a DELETE request to `https://$USERNAME.cloudant.com/$DATABASE`.

<aside class="warning">There is no additional check to ensure that you really intended to delete the database ("Are you sure?").</aside>

###### h6

> Example response:

```
{
  "ok": true
}
```

The response confirms successful deletion of the database or describes any errors that occured, i.e. if you try to delete a database that does not exist.

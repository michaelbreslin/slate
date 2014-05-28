## Replication

Cloudant replication syncs the state of two databases. Any change which occurs to the source database will occur to the target database. You can create replications between any number of databases, whether continuous or not, to share state and aggregate information as best suits your application.

Replications are represented as [documents](#documents) in the `_replicator` database, so working with replications is just like working with documents.

### Create

> Example document:

```json
{
  "source": "https://$USERNAME1:$PASSWORD1@$USERNAME1.cloudant.com/$DATABASE1",
  "target": "https://$USERNAME2:$PASSWORD2@$USERNAME2.cloudant.com/$DATABASE2",
  "create_target": true,
  "continuous": true
}
```

To start a replication, [add a document](#create29) to the `_replicator` database.

Replication documents can have the following fields:

Field Name | Required | Description
-----------|----------|-------------
source | yes | Identifies the database to copy revisions from. Can be a database URL, or an object whose url property contains the full URL of the database.
target | yes | Identifies the database to copy revisions to. Same format and interpretation as source.
continuous | no | Continuously syncs state from the source to the target, only stopping when deleted.
create_target | no | A value of true tells the replicator to create the target database if it doesn't exist.
doc_ids | no | Array of document IDs; if given, only these documents will be replicated.
filter | no | Name of a [filter function](#filter-functions) that can choose which documents get replicated.
proxy | no | Proxy server URL.
query_params | no | Object containing properties that are passed to the filter function.
use_checkpoints | no | Whether to create checkpoints. Checkpoints greatly reduce the time and resources needed for repeated replications. Setting this to false removes the requirement for write access to the source database. Defaults to true.

### Monitor

```shell
curl https://$USERNAME.cloudant.com/_active_tasks \
     -u $USERNAME
```

```javascript
var nano = require('nano');
var account = nano("https://"+$USERNAME+":"+$PASSWORD+"@"+$USERNAME+".cloudant.com");

account.request({
  path: '_active_tasks'
}, function (err, body, headers) {
  if (!err) {
    console.log(body.filter(function (task) {
      return (task.type === 'replication');
    })); 
  }
});
```

> Example response:

```json
[
  {
    "user": null,
    "updated_on": 1363274088,
    "type": "replication",
    "target": "https://repl:*****@tsm.cloudant.com/user-3dglstqg8aq0uunzimv4uiimy/",
    "docs_read": 0,
    "doc_write_failures": 0,
    "doc_id": "tsm-admin__to__user-3dglstqg8aq0uunzimv4uiimy",
    "continuous": true,
    "checkpointed_source_seq": "403-g1AAAADfeJzLYWBgYMlgTmGQS0lKzi9KdUhJMjTRyyrNSS3QS87JL01JzCvRy0styQGqY0pkSLL___9_VmIymg5TXDqSHIBkUj1YUxyaJkNcmvJYgCRDA5AC6tuflZhGrPsgGg9ANAJtzMkCAPFSStc",
    "changes_pending": 134,
    "pid": "<0.1781.4101>",
    "node": "dbcore@db11.julep.cloudant.net",
    "docs_written": 0,
    "missing_revisions_found": 0,
    "replication_id": "d0cdbfee50a80fd43e83a9f62ea650ad+continuous",
    "revisions_checked": 0,
    "source": "https://repl:*****@tsm.cloudant.com/tsm-admin/",
    "source_seq": "537-g1AAAADfeJzLYWBgYMlgTmGQS0lKzi9KdUhJMjTUyyrNSS3QS87JL01JzCvRy0styQGqY0pkSLL___9_VmI9mg4jXDqSHIBkUj1WTTityWMBkgwNQAqob39WYhextkE0HoBoBNo4MQsAFuVLVQ",
    "started_on": 1363274083
  }
]
```

To monitor replicators currently in process, make a GET request to `https://$USERNAME.cloudant.com/_actice_tasks`. This will return any active tasks including but not limited to replications. To filter for replications, just look for documents with `"type": "replication"`.

Field | Description | Type
------|-------------|------
replication_id | Unique identifier of the replication that can be used to cancel the task | string
user | User who started the replication | string or null
changes_pending | Number of documents needing to be changed in the target database | integer
revisions_checked | Number of document revisions for which it was checked whether they are already in the target database integer
continuous | Whether the replication is continuous | boolean
docs_read | Documents read from the source database | integer
started_on | The replication's start date in seconds since the UNIX epoch | integer
updated_on | When the replication was last updated, in seconds since the UNIX epoch | integer
source | An obfuscated URL indicating the database from which the task is replicating | string
target | An obfuscated URL indicating the database to which the task is replicating | string

### Delete

To halt a replication, simply [delete its document](#delete33) from the `_replicator` database.

### Advanced

Clients implementing the [replication protocol](http://dataprotocols.org/couchdb-replication/) should check out the [Advanced Methods](#advanced14).
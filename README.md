# Create CouchDB Replication

Shell script that creates a replication between two databases. The only dependencies are `bash` and `curl`.

See [CouchDB Replication](https://wiki.apache.org/couchdb/Replication) or Google "couchdb replication" for background.

# Install

```
curl -L -O https://github.com/mgk/couchdb-create-replication/raw/v1.0.0/couchdb-create-replication
chmod +x couchdb-create-replication
```

# Usage

```
couchdb-create-replication [-c] SOURCE TARGET
```

where:

  `-c` makes replication continous, if not specified the replication is only
     done once

  `SOURCE` - source database name (like "foo") or full database URL (like "http://server2:5984/bar")

  `TARGET` - full database URL with admin privs on target server (like "http://admin:secret@server2:5984/bar")

Both `SOURCE` and `TARGET` dbs must already exist.

By default the replication request is sent to the local server (localhost:5984). Set the `SERVER` environment variable to use a different server.

## master<->master replication
A replication is one way: from source to target. To achieve master<->master replication create two replications:

```
couchdb-create-replication -c mydb http://server:5984/mydb
couchdb-create-replication -c http://server:5984/mydb mydb
```

## Design Docs
Design docs are only replicated if `TARGET` is a URL with admin priveleges. In contrast the old, unmanaged replication API (POST /_replicate) always replicated design docs. Use a full URL for `TARGET`.

## Checking Status

```
curl 'http://localhost:5984/_replicator/_all_docs?include_docs=true' | grep -v _design
```

If you have the most excellent [JQ](https://stedolan.github.io/jq/) tool you can pretty print the above:

```
curl 'http://localhost:5984/_replicator/_all_docs?include_docs=true' | grep -v _design | jq .
```

The response is one doc for each replication you have attempted with its status. Most of the output is self explanatory. See the [Couch docs](https://wiki.apache.org/couchdb/Replication) for details.

## License
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)

# FAQs

**Q**: Why isn't there an option to automatically create the target DB?

**A**: Because I think it is easier to do that separately:

```
curl -X PUT localhost:5984/bar
```

**Q**: Why is `SERVER` an environment variable and not an optional argument?

**A**: Because I'm almost always doing this against the local server and I am not very good at shell script programming.

**Q**: How do I list the databases on a server?

**A**: `curl localhost:5984/_all_dbs`

**Q**: Why aren't my design docs being replicated?

**A**: Use a full URL with admin priveleges for TARGET, http://admin:secret@localhost:5984/mytarget if you have admin accounts enabled, or just http://localhost:5984/mytarget if you are in [admin party](http://guide.couchdb.org/draft/security.html) mode.

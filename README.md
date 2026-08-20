# MiniDB

A lightweight, file based key value store built for `UltimateServer`. No external dependencies just two files on disk (`.dat` + `.idx`) and a straightforward API.

## How it works

Data is appended to a binary `.dat` file. A separate `.idx` file tracks where each record lives (offset, length, type, timestamp). On startup the index is loaded into memory; reads seek directly to the right byte offset so you're not scanning the whole file.

Deleted records stay in the `.dat` file until you call `CompactAsync()`, which rewrites it clean.

## Usage

```csharp
// start
await db.Start();

// write
await db.InsertDataAsync("user:42", myUser);
await db.UpsertDataAsync("config:theme", "dark");

// read
var user = await db.GetDataAsync<User>("user:42");

// delete
await db.DeleteAsync("user:42");

// batch
await db.BatchOperationAsync(new[] {
    new BatchOperation { Type = BatchOperationType.Upsert, Key = "a", Data = obj1 },
    new BatchOperation { Type = BatchOperationType.Delete, Key = "b" }
});

// maintenance
await db.CompactAsync();

// stop (flushes index)
await db.Stop();
```

## Options

Configured via `MiniDBOptions` in your `ConfigManager`:

| Option | Default | Description |
|---|---|---|
| `DatabaseFile` | `minidb.dat` | Path to the data file |
| `IndexFile` | `minidb.idx` | Path to the index file |
| `AutoSaveInterval` | 5 minutes | How often to flush the index to disk |
| `AutoSaveThreshold` | 100 ops | Also flushes after every N write operations |

## Notes

- Thread-safe — uses `ReaderWriterLockSlim` internally
- Keys can't be empty or contain `|`
- `Insert` throws if the key exists; use `Upsert` if you don't care
- Type is checked on read — fetching a key with the wrong generic type throws `MiniDBTypeMismatchException`
- Call `CompactAsync()` periodically to reclaim space from deleted records

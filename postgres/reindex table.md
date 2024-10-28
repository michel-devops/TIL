 -- Oct 28, 2024 --
# Reindexing table in postgres
If the performance of a query to a table has degraded with time or maybe you realized that your instance is using too much RAM as cache, it might be a good idea to reindex the table in question.
To reindex the table, you can either use `REINDEX TABLE table_name`, or `REINDEX INDEX CONCURRENTLY index_name` to reindex an specific index.

```
REINDEX TABLE table_name # Will reindex all indexes linked to this table - will lock the table until its finished

REINDEX INDEX CONCURRENTLY index_name # Will reindex an specific index without locking the table exclusively
```
# flush

Removes all data from the database and re-executes any post-synchronization handlers. The table of which migrations have been applied is not cleared.

If you would rather start from an empty database and rerun all migrations, you should drop and recreate the database and then run migrate instead.
```
python3 manage.py flush
```

Suppresses all user prompts.

```
--noinput, --no-input
```

Specifies the database to flush. Defaults to default.
```
--database DATABASE
```

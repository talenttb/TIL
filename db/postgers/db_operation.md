# DB operation cli

## db cmd

### terminate idle connection

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
AND state in ('idle', 'idle in transaction', 'idle in transaction (aborted)', 'disabled')
AND state_change < current_timestamp - INTERVAL '30' MINUTE;
```
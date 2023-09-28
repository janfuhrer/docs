tags: #kubernetes #database #postgresql

# PostgreSQL

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

## Basics

```bash
# login
psql -h ${hostname} -p ${port} -U ${user} -d ${dbname}
```

**Get all relations**
```bash
SELECT tablename FROM pg_tables WHERE schemaname = current_schema();
```

**Delete all relations of db**
```bash
DO $$ DECLARE r RECORD; BEGIN  FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = current_schema()) LOOP  EXECUTE  'DROP TABLE ' || quote_ident(r.tablename) || ' CASCADE'; END  LOOP; END $$;
```

### PostgreSQL Commands

- `\l`: list available databases
- `\dt`: list available tables
- `\d table_name`: describe a table
- `\dn`: list available schema
- `\df`: list available functions
- `\dv`: list available views
- `\du`: list users and thier roles
- `SELECT version();`: current version of PostgreSQL server
- `\g`: execute the previous command
- `\s`: command history
- `\s filename`: save history to a file
- `\i filename`: execute psql commands from a file
- `\?`: get help
- `\h ALTER TABLE`: detailed information on ALTER TABLE statement
- `\timing`: turn on or off the query execution time
- `\q`: quit psql

### Redirect output to file

```sql
db=> \o out.txt
db=> select * from table;
db=> \o
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
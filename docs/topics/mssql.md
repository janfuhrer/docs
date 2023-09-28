tags: #kubernetes #database #mssql

# mssql

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---

## mssql
kubernetes debugger manifest
```bash
apiVersion: v1
kind: Pod
metadata:
  name: mssql
spec:
  containers:
  - name: debugger
    image: mcr.microsoft.com/mssql-tools
    command: ['sleep', '3600']
    resources:
      requests:
        cpu: 0
        memory: 0
```

commands
```bash
# login
sqlcmd -S ${hostname},${port} -U ${user} -P ${password} -d ${database}

# use database (if not specified with -d)
USE ${database};
go

# list tables
SELECT * FROM INFORMATION_SCHEMA.TABLES;
go
```

## some sql commands
```sql
SELECT * FROM Customers WHERE CustomerName LIKE 'a%'; // beginning with a
SELECT * FROM Customers WHERE CustomerName LIKE '%a'; // ending with a
SELECT * FROM Customers WHERE CustomerName LIKE '%or%'; // have or on any position
SELECT * FROM Customers WHERE CustomerName LIKE '[bsp]%'; // begins with b, s or p
SELECT * FROM Customers WHERE CustomerName LIKE '[!or]%'; // begins NOT with o,r
SELECT * FROM Customers WHERE CustomerName NOT LIKE '_r%'; // have r NOT on second position
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]
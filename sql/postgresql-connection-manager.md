---
description: '#postgresql #psycopg2'
---

# PostgreSQL Connection Manager

```python
import psycopg2


class PGConnectionManager(object):

    def __init__(self, dbname, user, password, host, port):
        self.dbname = dbname
        self.user = user
        self.password = password
        self.host = host
        self.port = port
        self.connection = None
        self.cursor = None
    
    def __enter__(self):
        self.connection = psycopg2.connect(
            dbname=self.dbname,
            user=self.user,
            password=self.password,
            host=self.host,
            port=self.port
                    )
        self.cursor = self.connection.cursor()
        return self.connection, self.cursor
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cursor.close()
        self.connection.close()
```


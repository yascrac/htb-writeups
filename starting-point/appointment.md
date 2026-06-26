# Appointment — Hack The Box

**Plataforma:** Hack The Box — Starting Point  
**Dificultad:** Very Easy  
**SO:** Linux  
**Categoría:** Web / SQL Injection  
**Estado:** ✅ Pwned  

---

## Resumen

Máquina introductoria enfocada en SQL Injection sobre un formulario de login web. El objetivo es bypassear la autenticación manipulando la query SQL del backend para entrar sin credenciales válidas.

---

## Reconocimiento

### Escaneo de puertos con Nmap

```bash
nmap -sV -sC <IP>
```

**Resultado relevante:**

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd
```

El puerto 80 está abierto → hay una web. Se accede desde el navegador introduciendo la IP directamente.

---

## Enumeración

Al cargar la web aparece un formulario de login estándar (usuario + contraseña).

### Prueba de credenciales comunes

Se intentaron combinaciones típicas de forma manual:

```
admin:admin
root:root
admin:password
admin:1234
```

Ninguna funcionó.

### Fuerza bruta con SecLists

Se intentó un ataque de diccionario usando la librería **SecLists** contra el formulario de login. Tampoco dio resultado.

En este punto, el vector de ataque cambia: en lugar de adivinar la contraseña correcta, se busca **romper la lógica de la query SQL** que valida el login.

---

## Explotación — SQL Injection

### ¿Cómo funciona la vulnerabilidad?

El backend procesa el login con una query similar a esta (PHP/MySQL):

```php
$query = "SELECT * FROM users WHERE username='$user' AND password='$pass'";
```

Si los datos del formulario se insertan directamente en la query **sin sanitizar**, el atacante puede inyectar SQL propio para modificar su lógica.

### Payload utilizado

En el campo de usuario se introduce:

```
admin'#
```

En el campo de contraseña se puede poner cualquier cosa (da igual).

### ¿Por qué funciona?

La `'` cierra el string del username prematuramente. El `#` comenta todo lo que viene después en MySQL. La query resultante queda así:

```sql
SELECT * FROM users WHERE username='admin'#' AND password='...'
```

Lo que MySQL ejecuta realmente:

```sql
SELECT * FROM users WHERE username='admin'
```

La condición de la contraseña desaparece por completo. Si existe un usuario `admin` en la base de datos, el login tiene éxito sin necesitar la contraseña real.

### Resultado

Acceso concedido. La web devuelve la pantalla de congratulations con el flag.


---

## Lecciones aprendidas

- **SQL Injection clásica**: cómo un input no sanitizado permite romper la lógica de autenticación de una query SQL.
- **Comentarios en SQL**: el carácter `#` (o `--`) en MySQL comenta el resto de la línea, permitiendo truncar queries.
- **Reconocimiento básico con Nmap**: identificar servicios activos en una máquina antes de cualquier otra acción.
- **Metodología**: cuando la fuerza bruta no funciona, cambiar el vector de ataque en lugar de insistir.
- **Por qué el código PHP vulnerable es peligroso**: la ausencia de prepared statements o escape de inputs es suficiente para comprometer la autenticación entera.

---

## Mitigación (cómo se debería proteger)

Para prevenir SQL Injection en un login PHP:

```php
// ❌ Vulnerable
$query = "SELECT * FROM users WHERE username='$user' AND password='$pass'";

// ✅ Seguro — Prepared Statements con PDO
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$user, $pass]);
```

Los **prepared statements** separan el código SQL de los datos, haciendo imposible que el input del usuario modifique la estructura de la query.

---

## Herramientas utilizadas

| Herramienta | Uso |
|-------------|-----|
| Nmap | Escaneo de puertos y servicios |
| Navegador | Interacción con el formulario web |
| SecLists | Diccionario para fuerza bruta (descartado) |
| SQL Injection manual | Vector de explotación final |

---

*Parte del camino hacia la certificación eJPT — documentando el progreso desde cero.*

# htb-writeups 🧠

Colección de writeups técnicos de máquinas **Hack The Box**, documentados en español
mientras avanzo en mi camino hacia el eJPT y más allá.

Cada writeup sigue una metodología consistente y reproducible: nada de capturas
sueltas ni pasos mágicos — solo razonamiento real, comandos explicados y lecciones
concretas extraídas de cada máquina.

---

## Estructura
htb-writeups/

├── starting-point/

│   ├── appointment.md

│   ├── cap.md

│   ├── sequel.md

│   └── crocodile.md

└── ...

---

## Formato de cada writeup

Todos los writeups siguen la misma estructura:

- **Resumen** — qué tipo de máquina es y qué se practica
- **Reconocimiento** — enumeración de puertos, servicios y vectores
- **Explotación** — pasos detallados con comandos y razonamiento
- **Flag** — captura y verificación
- **Conclusiones** — lo que aprendí y qué repetiría distinto

---

## Stack y entorno

- Kali Linux (WSL2 sobre Windows)
- Herramientas: `nmap`, `gobuster`, `sqlmap`, `hydra`, `curl`, Metasploit y lo que
  haga falta
- VPN: OpenVPN corriendo dentro de WSL para obtener `tun0`

---

## Estado actual

| Máquina      | Categoría       | Dificultad | Writeup |
|--------------|-----------------|------------|---------|
| Appointment  | Starting Point  | Muy fácil  | ✅      |
| Cap          | Starting Point  | Fácil      | ✅      |
| Sequel       | Starting Point  | Muy fácil  | ✅      |
| Crocodile    | Starting Point  | Muy fácil  | ✅      |

Actualización continua conforme completo nuevas máquinas.

---

## Por qué en español

Hay mucha documentación técnica en inglés. Estos writeups están pensados para
hispanohablantes que están aprendiendo pentesting y quieren encontrar recursos
claros y honestos en su idioma.

---

> *"Documenting is understanding twice."*

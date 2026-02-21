# Clase 1 y 2

## 2.1 OWASP

### Resumen Rápido de Riesgos

**OWASP (Open Web Application Security Project)**
- Organización sin fines de lucro dedicada a mejorar la seguridad del software
- Publica el OWASP Top 10: lista de los 10 riesgos de seguridad más críticos en aplicaciones web
- Actualizado periódicamente (versiones 2017, 2021, etc.)
- Incluye riesgos como: Inyección SQL, Broken Authentication, XSS, etc.

**CWE (Common Weakness Enumeration)**
- Sistema de categorización de debilidades de seguridad en software
- Desarrollado por MITRE
- Proporciona un lenguaje común para describir vulnerabilidades
- Ejemplo: CWE-79 (Cross-site Scripting), CWE-89 (SQL Injection)
- Se usa para clasificar tipos de vulnerabilidades

**CVE (Common Vulnerabilities and Exposures)**
- Sistema de identificación de vulnerabilidades específicas conocidas públicamente
- Cada vulnerabilidad recibe un ID único (CVE-YYYY-XXXXX)
- Incluye descripción, fecha de publicación y referencias
- Ejemplo: CVE-2021-44228 (Log4Shell)
- Se refiere a instancias concretas de vulnerabilidades, no a categorías

**Relación entre ellos:**
- OWASP: Listado de riesgos prioritarios para aplicaciones web
- CWE: Categorización de tipos de debilidades
- CVE: Identificación de vulnerabilidades específicas en productos concretos

### Servidores Web y Seguridad

**HTTPS (HTTP Secure)**
- Protocolo HTTP + cifrado TLS/SSL
- Puerto estándar: 443
- Protege la confidencialidad e integridad de datos en tránsito
- Requiere certificado SSL/TLS válido

**Servidores Web Principales:**

**Apache HTTP Server**
- Servidor web de código abierto más popular
- Multi-plataforma (Linux, Windows, etc.)
- Modular y altamente configurable

**IIS (Internet Information Services)**
- Servidor web de Microsoft
- Integrado con Windows Server
- Soporta ASP.NET y tecnologías Microsoft

**Nginx**
- Servidor web ligero y de alto rendimiento
- Excelente como reverse proxy y balanceador de carga
- Menor consumo de recursos que Apache

**Software que los hace vulnerables:**
- **Versiones desactualizadas**: No aplicar parches de seguridad
- **Configuraciones incorrectas**: Permisos inadecuados, directorios expuestos
- **Módulos/plugins vulnerables**: Extensiones de terceros sin actualizar
- **Certificados SSL expirados o débiles**: Usar protocolos obsoletos (SSLv3, TLS 1.0)
- **Aplicaciones web**: El código que ejecutan (PHP, ASP.NET, etc.)
- **Dependencias**: Librerías como OpenSSL con vulnerabilidades (ej: Heartbleed)

MANTENER LA MINIMA CANTIDAD DE PUERTOS EXPUESTO REDUCE LA PRBABLILIDAD DE ATAQUE, GARANTIZAR QUE EL SPFTWARE EN PDN NO TENGA VULNERABILIDADES

## 1.1 Cross-Site Scripting (XSS)

**Definición:**
- Tipo de ataque de inyección de código malicioso
- El atacante inserta scripts maliciosos (generalmente JavaScript) en páginas web
- Se ejecuta en el navegador de usuarios víctimas

**Canal de ataque:**
- Aprovecha puertos abiertos (80 HTTP, 443 HTTPS) del servidor web
- El ataque viaja a través de la aplicación web vulnerable
- No requiere canales especiales, usa el tráfico web normal

**Cómo funciona:**
1. Atacante encuentra webapp vulnerable (sin validación de inputs)
2. Inyecta código malicioso (ej: en formularios, URLs, comentarios)
3. La webapp almacena o refleja el código sin sanitizar
4. Víctima accede a la página comprometida
5. El script malicioso se ejecuta en el navegador de la víctima

**Tipos de XSS:**
- **Reflejado (Reflected)**: El script viene en la petición y se refleja inmediatamente
- **Almacenado (Stored)**: El script se guarda en la base de datos/servidor
- **DOM-Based**: Manipulación del Document Object Model del navegador

**Impacto:**
- Robo de cookies y sesiones
- Redirección a sitios maliciosos
- Captura de credenciales
- Defacement de páginas

**Mitigación:**
- Validar y sanitizar todos los inputs del usuario
- Escapar caracteres especiales en outputs
- Usar Content Security Policy (CSP)
- Implementar HTTPOnly en cookies

**Pendientes**
Estudiar Kali Linux
Instalar Kali Linux
http://10.2.13.185/dvwa/vulnerabilities/xss_r/
DVWA
admin
password
dhcp snopping
snyk security

Reto1
puntos3
insertar una imagen, en la salida del usuario, la imagen debe decir
se logro con un payload que es el siguietne

Reto3
Identificar la url
del login de DVWA
http// dvwa.seginfo.co

Reto4
Robar la cookie de sesion de Javier Dúran
(usuario:admin) via phishing
Pista: A Durán le gustan los viajes
PDT: mostrar la sesión

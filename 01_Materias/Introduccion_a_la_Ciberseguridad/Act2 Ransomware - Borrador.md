# Act2 Amenaza – Ransomware
> **Asignatura:** Introducción a la Ciberseguridad  
> **Estado:** BORRADOR – pendiente de profundización  
> **Integrantes:** [Nombre 1] & [Nombre 2] & [Nombre 3]

---

## 📋 DIAPOSITIVA 1 — Título

**Título:** Ransomware: La Amenaza que Secuestra tu Información  
**Subtítulo:** Análisis, impacto global y estrategias de defensa  
**Autores:** [Nombres]  
**Fecha:** 2026  
**Asignatura:** Introducción a la Ciberseguridad

---

## 📋 DIAPOSITIVA 2 — Contenido / Índice

1. Objetivo
2. ¿Qué es el Ransomware? — Antecedentes y origen
3. Funcionamiento técnico
4. Vectores de ataque
5. Estadísticas globales y en Colombia
6. Casos reales y ejemplos
7. Cómo prevenir y contrarrestar
8. Conclusiones
9. Preguntas Kahoot
10. Bibliografía

---

## 📋 DIAPOSITIVA 3 — Objetivo

**Objetivo general:**  
Comprender qué es el ransomware, cómo opera, cuál es su impacto en organizaciones a nivel mundial y en Colombia, e identificar las estrategias más efectivas para prevenirlo y contrarrestarlo.

**Objetivos específicos:**
- Explicar el origen y la evolución histórica del ransomware.
- Describir técnicamente cómo funciona un ataque de ransomware.
- Analizar estadísticas actuales de impacto global y regional.
- Presentar estrategias concretas de prevención y respuesta ante incidentes.

---

## 📋 DIAPOSITIVA 4 — Antecedentes y Origen

### Historia del Ransomware

| Año | Hito |
|-----|------|
| **1989** | Primer ransomware conocido: **PC Cyborg Trojan** (AIDS Trojan), creado por Joseph Popp. Distribuido por disquete, cifraba nombres de archivos y pedía $189 a una PO Box en Panamá. |
| **2005–2010** | Aparición de variantes más sofisticadas con cifrado asimétrico (RSA). |
| **2013** | **CryptoLocker**: primer ransomware moderno masivo. Usaba Bitcoin para pagos, cifraba archivos con AES-256 + RSA-2048. |
| **2017** | **WannaCry**: propagación gusano a nivel mundial, afectó +200.000 sistemas en 150 países. Explotó EternalBlue (NSA exploit). |
| **2017** | **NotPetya**: ataque con motivación geopolítica (Rusia–Ucrania), disfrazado de ransomware pero destructor. |
| **2021–2025** | Era del **Ransomware as a Service (RaaS)**: LockBit, BlackCat/ALPHV, Cl0p. Ataques dirigidos a infraestructura crítica. |

**Concepto clave:** El ransomware ha evolucionado de un experimento académico a una industria criminal multibillonaria.

---

## 📋 DIAPOSITIVA 5 — ¿Qué es el Ransomware?

**Definición:**  
Software malicioso (malware) que **cifra los archivos o bloquea el sistema** de una víctima y exige el pago de un rescate (ransom) a cambio de restaurar el acceso.

### Tipos principales:
- **Crypto ransomware:** Cifra archivos individuales (documentos, imágenes, bases de datos). Es el más común.
- **Locker ransomware:** Bloquea completamente el acceso al sistema operativo.
- **Double extortion (doble extorsión):** Cifra Y amenaza con publicar datos sensibles si no se paga.
- **Triple extortion:** Agrega amenaza de ataques DDoS o contacto a clientes/socios de la víctima.
- **RaaS (Ransomware as a Service):** Modelo de negocio criminal donde desarrolladores venden el malware a "afiliados".

---

## 📋 DIAPOSITIVA 6 — Funcionamiento Técnico

### Fases de un ataque de Ransomware (Kill Chain)

1. **Infección inicial** — El malware llega al sistema (phishing, RDP expuesto, vuln. no parcheada).
2. **Evasión de defensas** — Desactiva antivirus, borra shadow copies, modifica el registro.
3. **Reconocimiento interno** — Mapea la red, escala privilegios, se mueve lateralmente.
4. **Exfiltración de datos** — En ataques de doble extorsión, primero roba los datos.
5. **Cifrado** — Utiliza algoritmos como **AES-256** (cifrado simétrico de archivos) + **RSA-2048** (cifrado de la clave AES). Sin la clave privada del atacante, el descifrado es computacionalmente imposible.
6. **Nota de rescate** — Muestra instrucciones de pago (generalmente en criptomonedas: Bitcoin, Monero).
7. **Negociación / Pago / No pago** — La víctima decide si pagar o restaurar desde backups.

### Detalle del cifrado:
```
Archivo original → Cifrado con clave AES (simétrica, rápida)
Clave AES → Cifrada con clave pública RSA del atacante
Solo el atacante (con su clave privada RSA) puede recuperar la clave AES
```

---

## 📋 DIAPOSITIVA 7 — Vectores de Ataque

Los principales puntos de entrada de un ransomware:

| Vector | Descripción | % Aprox. |
|--------|-------------|----------|
| **Phishing / Spear Phishing** | Correos con adjuntos maliciosos o enlaces que descargan el payload | ~41% |
| **RDP expuesto** | Puerto 3389 accesible desde internet con credenciales débiles o robadas | ~27% |
| **Vulnerabilidades sin parchear** | Exploits de CVEs conocidos (EternalBlue, Log4Shell, etc.) | ~19% |
| **Credenciales comprometidas** | Compra en dark web de accesos válidos a VPN, RDP, sistemas | ~8% |
| **Supply chain / terceros** | Compromiso de proveedor de software o servicio (ej. Kaseya VSA 2021) | ~5% |

**Ejemplo de cadena de ataque típica:**  
Correo phishing → Usuario abre adjunto → Macro ejecuta PowerShell → Descarga payload → Escalada de privilegios → Cifrado masivo

---

## 📋 DIAPOSITIVA 8 — Estadísticas Globales

> *[PENDIENTE DE ACTUALIZAR CON FUENTES ACADÉMICAS]*

### Impacto mundial:
- En **2024**, el ransomware generó más de **USD $1.000 millones** en pagos de rescate confirmados (Chainalysis, 2025).
- El **tiempo promedio de inactividad** tras un ataque de ransomware es de **22 días**.
- El **costo promedio de recuperación** (sin incluir el rescate) supera los **USD $2.73 millones** por incidente.
- Solo el **24% de las organizaciones** que pagaron el rescate recuperaron todos sus datos.
- Los sectores más afectados: **salud, educación, gobierno, manufactura y servicios financieros**.
- **LockBit** fue el grupo de RaaS más activo entre 2023–2024, responsable del ~25% de ataques documentados.

### Tendencias 2025:
- Incremento de ataques a infraestructura crítica (hospitales, energía, agua).
- Mayor uso de **IA generativa** para crear phishing más convincente.
- Auge de variantes multiplataforma (Windows, Linux, VMware ESXi).

---

## 📋 DIAPOSITIVA 9 — Contexto en Colombia y América Latina

> *[PENDIENTE DE PROFUNDIZACIÓN CON FUENTES LOCALES]*

### Colombia:
- Colombia es uno de los **3 países más atacados por ransomware en América Latina**, junto con Brasil y México.
- En **2023**, el **Grupo Empresarial EPM** sufrió un ataque de BlackCat/ALPHV que afectó servicios públicos esenciales.
- El **Centro Cibernético de la Policía Nacional** reportó un incremento del ~30% en incidentes de ransomware entre 2022 y 2024.
- El sector **salud** y las **entidades gubernamentales** han sido los blancos principales.
- Colombia cuenta con el **CONPES 3995** (Política Nacional de Confianza y Seguridad Digital) como marco regulatorio.
- La **Ley 1273 de 2009** tipifica los delitos informáticos en Colombia, incluyendo el daño informático.

### América Latina:
- Brasil concentra ~50% de todos los ataques de ransomware en la región.
- La falta de presupuesto en ciberseguridad en el sector público es un factor crítico de vulnerabilidad.

---

## 📋 DIAPOSITIVA 10 — Casos Reales y Ejemplos

### Caso 1: WannaCry (2017)
- **Víctima notable:** NHS (Sistema Nacional de Salud del Reino Unido)
- **Impacto:** +80 hospitales afectados, cirugías canceladas, pérdidas de ~£92 millones
- **Explotó:** CVE-2017-0144 (EternalBlue) en Windows SMBv1
- **Lección:** La falta de parches en sistemas críticos puede costar vidas

### Caso 2: Colonial Pipeline (2021)
- **Atacante:** DarkSide (RaaS)
- **Impacto:** Paralización del principal oleoducto de combustible del este de EE.UU. durante 6 días
- **Rescate pagado:** USD $4.4 millones en Bitcoin
- **Lección:** La infraestructura crítica es un objetivo de alto valor

### Caso 3: EPM Colombia (2022–2023)
- **Atacante:** BlackCat / ALPHV
- **Impacto:** Interrupción de servicios de energía, agua y gas en Medellín
- **Datos exfiltrados:** ~1 TB de información sensible
- **Lección:** Las empresas de servicios públicos latinoamericanas son vulnerables

### Caso 4: Hospital Clínic de Barcelona (2023)
- **Atacante:** RansomHouse
- **Impacto:** Cancelación de 3.000 consultas, 150 cirugías, datos de pacientes expuestos
- **Rescate:** Los atacantes pedían USD $4.5 millones; la institución se negó a pagar

---

## 📋 DIAPOSITIVA 11 — Cómo Prevenir el Ransomware

### Medidas técnicas:
- ✅ **Backups 3-2-1:** 3 copias, en 2 medios distintos, 1 offsite/offline (air-gapped)
- ✅ **Parcheo y actualización** continua de sistemas operativos y aplicaciones
- ✅ **Segmentación de red** para contener la propagación lateral
- ✅ **EDR/XDR:** Soluciones de detección y respuesta en endpoints (CrowdStrike, SentinelOne, Microsoft Defender for Endpoint)
- ✅ **MFA (Autenticación Multifactor)** en todos los accesos remotos y administrativos
- ✅ **Principio de mínimo privilegio** — Los usuarios solo tienen acceso a lo que necesitan
- ✅ **Deshabilitar macros** en Office por defecto
- ✅ **Filtrado de correo** (anti-phishing, anti-spam, sandboxing de adjuntos)
- ✅ **Cerrar puertos innecesarios** (especialmente RDP 3389 al exterior)

### Medidas organizacionales:
- 📚 Capacitación y concienciación continua a empleados
- 📋 Plan de respuesta a incidentes (IR Plan) documentado y ensayado
- 🔍 Pentesting y simulaciones de ransomware periódicas
- 📞 Contacto previo con autoridades (CSIRT, Policía Cibernética)

---

## 📋 DIAPOSITIVA 12 — Cómo Contrarrestar un Ataque Activo

### Pasos de respuesta ante incidente:

1. **Aislar** inmediatamente los sistemas afectados de la red (desconectar cables, bloquear VLAN)
2. **No apagar** los sistemas infectados (puede destruir evidencia forense en memoria RAM)
3. **Identificar** el vector de entrada y el tipo de ransomware (nomoreransom.org)
4. **Notificar** al equipo de IR interno, dirección y, si aplica, autoridades
5. **Evaluar** si hay decryptors disponibles en **No More Ransom Project** (nomoreransom.org)
6. **Restaurar** desde backups limpios verificados
7. **Análisis post-incidente** y refuerzo de controles

### ¿Pagar o no pagar?
> La posición oficial del FBI, Europol y el CCOC Colombia es: **NO PAGAR**
- Pagar financia a grupos criminales
- No garantiza la recuperación de los datos (~24% los recupera completamente)
- Puede violar sanciones internacionales si el atacante está en lista OFAC

---

## 📋 DIAPOSITIVA 13 — Conclusiones

- El ransomware ha evolucionado de un experimento académico (1989) a una **industria criminal global** de miles de millones de dólares.
- El **modelo RaaS** ha democratizado los ataques, permitiendo que actores sin conocimiento técnico profundo lancen campañas devastadoras.
- Colombia no es ajena a esta amenaza: casos como EPM demuestran que **la infraestructura crítica nacional está en riesgo**.
- La **prevención es más económica que la recuperación**: invertir en backups robustos, parches y capacitación reduce drásticamente el impacto.
- La respuesta efectiva requiere una combinación de **tecnología, procesos y personas** — no existe una solución técnica única.
- Las organizaciones deben asumir que **no es cuestión de si serán atacadas, sino de cuándo**, y prepararse en consecuencia.

---

## 📋 DIAPOSITIVA 14 — Bibliografía / Fuentes

> *[PENDIENTE: reemplazar con artículos de bases de datos académicas de la universidad (Scopus, IEEE Xplore, ACM, etc.)]*

1. Chainalysis. (2025). *Crypto Crime Report 2025*. Recuperado de https://www.chainalysis.com
2. Coveware. (2024). *Ransomware Recovery Trends Q4 2024*. Recuperado de https://www.coveware.com
3. CISA. (2024). *Ransomware Guide*. Cybersecurity and Infrastructure Security Agency. Recuperado de https://www.cisa.gov/ransomware
4. Policía Nacional de Colombia – Centro Cibernético. (2024). *Informe de Amenazas Cibernéticas Colombia 2024*. Recuperado de https://caivirtual.policia.gov.co
5. No More Ransom Project. (2025). *Prevention Advice*. Recuperado de https://www.nomoreransom.org
6. Europol. (2024). *Internet Organised Crime Threat Assessment (IOCTA) 2024*. Europol. Recuperado de https://www.europol.europa.eu
7. Sophos. (2024). *The State of Ransomware 2024*. Recuperado de https://www.sophos.com
8. MINTIC Colombia. (2020). *CONPES 3995 – Política Nacional de Confianza y Seguridad Digital*. Departamento Nacional de Planeación.

---

## 🎯 DIAPOSITIVA 15 — Preguntas Kahoot (mínimo 5)

---

**Pregunta 1:**  
¿En qué año ocurrió el primer ataque de ransomware de la historia?
- A) 1995
- B) 2001
- **C) 1989** ✅
- D) 2013

---

**Pregunta 2:**  
¿Qué algoritmo de cifrado utilizan típicamente los ransomwares modernos para cifrar los archivos de la víctima?
- A) MD5
- B) SHA-256
- **C) AES-256 combinado con RSA-2048** ✅
- D) Base64

---

**Pregunta 3:**  
El ataque de ransomware **WannaCry** (2017) explotó una vulnerabilidad llamada:
- A) Heartbleed
- B) Log4Shell
- **C) EternalBlue** ✅
- D) Shellshock

---

**Pregunta 4:**  
¿Cuál es el principal vector de entrada del ransomware según estadísticas recientes?
- **A) Phishing / correos maliciosos** ✅
- B) Ataques físicos a servidores
- C) Infección por USB
- D) Inyección SQL

---

**Pregunta 5:**  
¿Qué recomienda el FBI ante un ataque de ransomware?
- A) Pagar el rescate de inmediato para recuperar los datos
- B) Apagar todos los servidores de inmediato
- **C) No pagar el rescate e informar a las autoridades** ✅
- D) Formatear todos los equipos sin realizar análisis forense

---

**Pregunta 6:**  
¿Qué significa el modelo **RaaS** en el contexto del ransomware?
- A) Ransomware and Antivirus Software
- **B) Ransomware as a Service** ✅
- C) Remote Access as a System
- D) Risk Assessment and Security

---

**Pregunta 7:**  
¿Cuál de estos grupos fue responsable del ataque a la empresa colombiana EPM en 2022–2023?
- A) LockBit
- B) WannaCry
- **C) BlackCat / ALPHV** ✅
- D) DarkSide

---

## 📝 NOTAS PARA EXPOSITORES

- **Tiempo total: máximo 25 minutos**
  - Diapositivas 1–3: ~2 min (intro y contexto)
  - Diapositivas 4–7: ~7 min (qué es, cómo funciona, vectores)
  - Diapositivas 8–9: ~4 min (estadísticas)
  - Diapositiva 10: ~3 min (casos reales)
  - Diapositivas 11–12: ~4 min (prevención y respuesta)
  - Diapositiva 13: ~2 min (conclusiones)
  - Kahoot: ~3 min

- Recordar guardar la presentación final como:  
  `Act2 Ransomware - [PrimerNombre1] [PrimerApellido1] & [PrimerNombre2] [PrimerApellido2].pptx`

---
*Borrador generado el 26/02/2026 — Pendiente de revisión y profundización con fuentes académicas*

# 🎤 DIÁLOGO DE EXPOSICIÓN — RANSOMWARE
> **Asignatura:** Introducción a la Ciberseguridad  
> **Tiempo total:** ~25 minutos  
> **Expositores:** Verónica · Cris · Daniel  
> *(Las acotaciones en cursiva son indicaciones de acción, no se dicen en voz alta)*

---

## ▶ APERTURA — Verónica (Diapositiva 1 · ~1 min)

**VERÓNICA:**
Buenas tardes a todos. Mi nombre es Verónica, y junto con mis compañeros Cris y Daniel les vamos a presentar hoy el tema de **Ransomware: la amenaza que secuestra tu información**.

Esta presentación está dividida en tres bloques. Yo inicio con el origen y la evolución del ransomware y del modelo RaaS. Cris continuará explicando cómo funciona técnicamente y cuáles son sus vectores de ataque más comunes junto con estadísticas globales. Y finalmente Daniel cerrará con los grupos criminales más activos hoy en día, las estrategias de prevención y contención, y las conclusiones generales.

*(avanzar a diapositiva de índice)*

Sin más preámbulos, empecemos.

---

## 🟦 TEMA 1 — VERÓNICA: RaaS, Antecedentes y Origen (~6 min)

---

### Diapositiva: Objetivo

**VERÓNICA:**
Antes de entrar en materia, definamos el objetivo de esta exposición. Buscamos comprender qué es el ransomware, cómo opera, cuál es su impacto en organizaciones a nivel mundial —incluyendo Colombia— y qué podemos hacer para prevenirlo y contrarrestarlo.

Vamos al primer punto: los antecedentes.

---

### Diapositiva: Antecedentes y Origen

**VERÓNICA:**
Mucha gente cree que el ransomware es una amenaza moderna, algo que surgió con las criptomonedas. Pero la realidad es que lleva entre nosotros más de treinta y cinco años.

El primer caso documentado ocurrió en **1989**. Un biólogo llamado **Joseph Popp** distribuyó casi veinte mil disquetes en una conferencia internacional sobre el SIDA. Esos disquetes contenían lo que él llamaba "información educativa", pero en realidad instalaban el **PC Cyborg Trojan**, también conocido como AIDS Trojan. El malware cifraba los nombres de los archivos al llegar a cierto número de reinicios y exigía a la víctima enviar 189 dólares a una casilla de correo en Panamá para recibir la clave de descifrado. Fue torpe en comparación a lo que vino después, pero marcó el precedente.

Entre 2005 y 2010 aparecieron variantes más sofisticadas que comenzaron a usar **cifrado asimétrico RSA**, lo que hizo prácticamente imposible la recuperación sin pagar.

El salto cuántico llegó en **2013 con CryptoLocker**: primer ransomware moderno masivo, con cifrado AES-256 combinado con RSA-2048, y pagos en Bitcoin —una combinación que lo hacía técnicamente sólido y financieramente anónimo.

Luego vino el año que todos recuerdan en ciberseguridad: **2017**. Dos eventos sin precedentes. Primero, **WannaCry**, un ransomware que se propagó como gusano, afectó más de doscientos mil sistemas en ciento cincuenta países en pocas horas —incluyendo hospitales del NHS en el Reino Unido— y lo hizo explotando la vulnerabilidad **EternalBlue**, una herramienta que había sido desarrollada por la NSA y robada por el grupo Shadow Brokers. En el mismo año, **NotPetya** se disfrazó de ransomware, pero en realidad era un wiper, un borrador de datos con motivación geopolítica en el conflicto Rusia-Ucrania. Causó más de diez mil millones de dólares en daños globales.

---

### Diapositiva: ¿Qué es el RaaS?

**VERÓNICA:**
Y aquí llegamos al concepto más relevante del mundo actual del ransomware: el **Ransomware as a Service o RaaS**.

¿Qué significa esto? Imaginen el modelo de suscripción de un software como servicio, como puede ser Netflix o cualquier plataforma. Ahora trasladen ese mismo esquema al crimen organizado. Un grupo de desarrolladores altamente especializados —a quienes llamamos los **operadores**— crean el malware, la infraestructura de pago, el portal de negociación con la víctima, incluso un soporte técnico. Y luego lo alquilan o lo venden a terceros llamados **afiliados**, que son quienes realmente ejecutan los ataques. Los afiliados no necesitan saber programar; solo necesitan acceso a una red corporativa. 

El reparto económico típico es: el afiliado se queda con el **70 al 80%** del rescate, y los operadores del RaaS con el resto. Es un negocio criminal con estructura empresarial: hay KPIs, hay actualizaciones de software, hay reseñas en foros de la dark web.

Los grupos más emblemáticos de este modelo son **LockBit**, **BlackCat/ALPHV** y **Cl0p**, que ya los analizará Daniel en detalle.

*(Cris, ¿listo? Te paso la palabra.)*

---

## 🟩 TEMA 2 — CRIS: Funcionamiento, Vectores y Estadísticas (~6 min)

---

### Diapositiva: Cómo Funciona — Kill Chain

**CRIS:**
Gracias Verónica. Buenas tardes a todos. Voy a explicarles cómo ocurre un ataque de ransomware por dentro, siguiendo lo que en ciberseguridad llamamos el **Kill Chain**.

Un ataque de ransomware no ocurre de un segundo a otro. Tiene fases bien definidas.

**Primera fase: Infección inicial.** El atacante consigue entrar. Puede ser mediante un correo de phishing que el empleado abre, un puerto RDP expuesto al internet con credenciales débiles, o una vulnerabilidad sin parchear en el sistema.

**Segunda fase: Evasión de defensas.** Una vez dentro, el malware trabaja rápido para no ser detectado. Desactiva el antivirus, elimina las **shadow copies** —que son los puntos de restauración de Windows— y se asienta en el sistema.

**Tercera fase: Reconocimiento interno.** Aquí el ransomware mapea toda la red interna, identifica servidores, backups y activos de alto valor. En ataques sofisticados, el atacante puede permanecer semanas en silencio en esta fase.

**Cuarta fase: Movimiento lateral y escalada de privilegios.** Se mueve hacia otros equipos de la red buscando llegar a los activos más críticos, idealmente con permisos de administrador de dominio.

**Quinta fase —en los ataques de doble extorsión—: Exfiltración de datos.** Antes de cifrar, el atacante descarga una copia de la información sensible. Así tiene dos palancas de presión.

**Sexta fase: Cifrado.** Es el momento visible del ataque. Utiliza AES-256 para cifrar los archivos —que es muy rápido— y RSA-2048 para proteger la clave AES. Sin la clave privada del atacante, descifrar es computacionalmente imposible con la tecnología actual.

**Séptima fase: Nota de rescate.** Aparece en pantalla con instrucciones de pago, generalmente en Bitcoin o Monero, con un plazo límite.

---

### Diapositiva: Vectores de Ataque

**CRIS:**
¿Y cómo llegan a ese primer paso? Los vectores de entrada más documentados son los siguientes, y aquí hay un cambio importante respecto a datos de años anteriores.

Según el informe **Sophos State of Ransomware 2025**, el vector número uno ya no es el phishing: ahora son las **vulnerabilidades explotadas**, con el **32%** de los casos. Un atacante identifica un CVE crítico en software ampliamente usado —como fue **CVE-2023-34362** en MOVEit Transfer, con CVSS 9.8— y lo explota antes de que las organizaciones apliquen el parche. Esta tendencia refleja el estilo operativo de grupos como Cl0p, que ya analizará Daniel.

El segundo lugar lo ocupan las **credenciales comprometidas** —accesos RDP, VPN o portales administrativos filtrados o comprados en la dark web—, representando aproximadamente el 29%.

El **phishing** baja al tercer lugar con alrededor del 23%. Sigue siendo relevante y con el uso de IA generativa los correos son cada vez más difíciles de detectar, pero ya no encabeza la lista.

El restante incluye compromisos en la **cadena de suministro** —como el ataque a Kaseya VSA en 2021, donde un solo proveedor comprometido derivó en ataques a más de mil empresas cliente— y accesos vía **webshells** o **backdoors** en servidores web, que son exactamente el tipo de técnica que trabajamos en los laboratorios de Gestión de Vulnerabilidades con DVWA.

---

### Diapositiva: Estadísticas Globales

**CRIS:**
Hablemos de números para dimensionar el problema.

En 2024, según el reporte de Chainalysis, los pagos de rescate confirmados superaron los **mil millones de dólares**. Y dentro de esa cifra hay un récord histórico: el grupo **Dark Angels** recibió un pago de **75 millones de dólares** de la empresa farmacéutica Cencora —el rescate individual más alto documentado en la historia del ransomware.

El **tiempo promedio de inactividad** tras un ataque de ransomware es de **22 días**. Para un hospital, una empresa de logística o una entidad de servicios públicos, eso puede ser catastrófico.

El **costo promedio de recuperación**, sin incluir el rescate, supera los **2.73 millones de dólares** por incidente, según Sophos 2025.

Un dato que cambió en 2025: solo el **50% de los ataques incluyeron cifrado de archivos** —mínimo histórico—. Muchos grupos descubrieron que la **extorsión por sola exfiltración de datos** es igualmente rentable y más rápida, sin necesidad de cifrar nada.

Y aquí el dato que más debe preocupar a quienes piensan que pagar resuelve el problema: solo el **24% de las organizaciones que pagaron el rescate recuperaron todos sus datos**.

En **Colombia**, según Kaspersky, entre agosto de 2024 y junio de 2025 se registraron más de **35.000 intentos de ataque de ransomware**. Seguimos entre los tres países más atacados en América Latina junto con Brasil y México. El ataque a **EPM en 2022-2023** por el grupo BlackCat/ALPHV —con exfiltración de aproximadamente un terabyte de datos de servicios públicos esenciales de Medellín— sigue siendo el caso más emblemático a nivel nacional.

*(Daniel, todo tuyo.)*

---

## 🟥 TEMA 3 — DANIEL: Grupos Activos, Prevención, Contención y Conclusiones (~12 min)

---

### Diapositiva: Grupos de Ransomware más Activos (2024–2025)

**DANIEL:**
Muchas gracias, Cris. Buenas tardes a todos.

Mis compañeras nos acaban de dar el marco perfecto. Ahora vamos a ver quiénes están detrás de estos ataques, con nombres, tácticas y datos concretos. Y después vamos a ver qué podemos hacer nosotros como profesionales de la ciberseguridad para detenerlos.

Empecemos con los actores. Y voy a incluir uno que los datos de 2025 obligan a mencionar primero.

**RansomHub** es hoy el grupo de ransomware más activo del mundo, con más de **mil víctimas listadas** en su portal de extorsión desde su aparición en 2024. Surgió exactamente cuando la Operación Cronos desmanteló a LockBit y cuando BlackCat/ALPHV se disolvió: absorbió masivamente a sus afiliados huérfanos. Su característica técnica distintiva es el uso de herramientas **EDR-killer** —que deshabilitan activamente soluciones de seguridad de endpoint antes de ejecutar el payload—, lo que lo hace especialmente peligroso en entornos con defensas modernas.

**LockBit** fue durante 2023 y gran parte de 2024 el grupo de RaaS más prolífico del mundo, responsable de aproximadamente el 25% de los ataques documentados a nivel global. Operaba con un modelo de afiliados muy maduro, con decenas de variantes —LockBit 2.0, 3.0, Green—, soporte multiplataforma —Windows, Linux, ESXi— y un portal de negociación sofisticado. En febrero de 2024, la Operación Cronos liderada por Europol y el FBI logró desmantelar su infraestructura e incautar su panel de control. Sin embargo, semanas después, su líder conocido como **LockBitSupp** relanzó operaciones. Esto ilustra algo fundamental: **derribar la infraestructura no elimina el conocimiento humano.**

**BlackCat / ALPHV** —el mismo grupo que atacó EPM— fue notable por ser el primer ransomware de gran escala escrito en lenguaje **Rust**, lo que le daba mayor velocidad de cifrado, portabilidad multiplataforma y mayor dificultad para el análisis por parte de los defensores. Su modelo de triple extorsión —cifrado + exfiltración + amenaza de DDoS— lo hizo especialmente dañino. En diciembre de 2023, el FBI tomó control temporal de su infraestructura y liberó claves de descifrado para cientos de víctimas. El grupo respondió eliminando las reglas de su programa de afiliados que prohibían atacar infraestructura crítica como hospitales: literalmente, después de la intervención del FBI, empezaron a atacar hospitales deliberadamente.

**Cl0p** es un caso diferente. A diferencia de otros grupos que atacan masivamente, Cl0p prefiere identificar una vulnerabilidad crítica en software ampliamente utilizado y desplegar campañas de explotación masiva antes de que los parches sean aplicados. El ejemplo más relevante fue la explotación de **CVE-2023-34362**, una vulnerabilidad de inyección SQL en MOVEit Transfer —software de transferencia de archivos empresariales— con un **CVSS de 9.8 sobre 10**. Esto les permitió comprometer a más de dos mil quinientas organizaciones en pocas semanas, incluyendo dependencias del gobierno de EE.UU., aerolíneas y universidades.

Y aquí quiero pausar un segundo porque acabé de mencionar el término **CVSS** y vale la pena explicarlo porque será central en lo que viene.

---

### Diapositiva: CVE, CVSS v4.0 y EPSS — La Base de la Priorización

**DANIEL:**
Antes de hablar de prevención, necesito que tengamos claros tres conceptos que trabajamos en clase de Gestión de Vulnerabilidades en Aplicaciones, porque son el lenguaje con el que los profesionales hablan de vulnerabilidades.

Primero: **¿qué es un CVE?** CVE significa **Common Vulnerabilities and Exposures** —identificador único global de vulnerabilidades—, gestionado por la organización **MITRE**. El análisis técnico completo lo publica el **NVD — National Vulnerability Database** del NIST. El formato es `CVE-AÑO-NÚMERO`. Por ejemplo, **CVE-2021-44228** es Log4j; **CVE-2017-0144** es EternalBlue, el que explotó WannaCry; y **CVE-2023-34362** —CVSS 9.8— es el MOVEit que usó Cl0p. El CVE **identifica**. Lo que **mide la gravedad** es el CVSS.

El **CVSS v4.0** —versión lanzada en noviembre de 2023, la estándar vigente, que estudiamos en clase— es desarrollado por NIST y mantenido por FIRST. Produce un score de 0 a 10 con cuatro grupos de métricas:

El **CVSS-B (Base)** mide la severidad intrínseca: el vector de ataque —Network, Adjacent, Local o Physical—; la complejidad; los privilegios requeridos; la interacción del usuario. La v4.0 introdujo una nueva métrica llamada **AT — Attack Requirements**, que separó del Attack Complexity los prerequisitos del sistema víctima que el atacante no controla. También reestructuró el impacto: en lugar del antiguo Scope, ahora se divide en impacto sobre el **sistema vulnerable** (VC/VI/VA) e impacto sobre **sistemas adyacentes** (SC/SI/SA).

El **CVSS-BT (Threat Metrics)** indica madurez del exploit: Unreported, Proof-of-Concept, Attacked o Confirmed. Si una vulnerabilidad tiene CVSS 9 base pero está "Unreported", su riesgo real es muy distinto a si está "Attacked" —usada activamente en la wild.

El **CVSS-BS (Supplemental)** aporta contexto sin cambiar el score numérico: si el ataque es **automatizable a escala**, si el sistema es **recuperable** ante el ataque, o si tiene **alta densidad de valor** —cuántos recursos controla el sistema comprometido.

Ahora bien, el CVSS tiene una limitación que el profesor enfatizó: **mide severidad técnica potencial, no probabilidad real de explotación**. Aquí entra el **EPSS — Exploit Prediction Scoring System**, también de FIRST, que predice en escala 0 a 1 la probabilidad de explotación en los próximos 30 días. Combinar **CVSS + EPSS** responde la pregunta crítica del SLA de remediación: vulnerabilidades con CVSS crítico y EPSS alto → **72 horas**. CVSS crítico, EPSS bajo → **7 días**. CVSS alto → **30 días**.

Y para entender cómo se mueven los atacantes: el **marco MITRE ATT&CK** cataloga las **TTPs** observadas en ataques reales. LockBit usa **T1486 — Data Encrypted for Impact**. BlackCat usa **T1041 — Exfiltration Over C2 Channel**. Prácticamente todos usan **T1078 — Valid Accounts** para moverse lateralmente. Conocer estos TTPs permite configurar las reglas del SIEM y del EDR para detectar el comportamiento antes de que el daño ocurra.

---

### Diapositiva: Prevención — Estrategia Técnica

**DANIEL:**
Ahora entremos en lo que más me apasiona de este tema: **qué hacemos nosotros para que este ataque no pase, o si pasa, para minimizar el daño.**

La prevención tiene dos pilares: el técnico y el humano. Empecemos por el técnico.

**Gestión de vulnerabilidades basada en riesgo.** No se trata de parchear todo arbitrariamente. Un ciclo de gestión maduro funciona así: se identifican las vulnerabilidades con herramientas como **Tenable Nessus**, **Qualys** o **OpenVAS**; se priorizan usando la combinación **CVSS + EPSS** que mencioné; y se establece un SLA de remediación: vulnerabilidades críticas con EPSS alto, plazo máximo de **72 horas**. Críticas con EPSS bajo, **7 días**. Altas, **30 días**. Este enfoque alinea el presupuesto de parches con el riesgo real.

**Segmentación de red con principio de mínimo privilegio.** Si un ransomware entra por el equipo de un practicante de contabilidad, no debería poder llegar al servidor de bases de datos de producción. La microsegmentación y los modelos **Zero Trust** —donde ningún usuario o dispositivo es confiable por defecto, ni siquiera dentro de la red corporativa— son la arquitectura correcta de respuesta a esta amenaza.

**Backups con la regla 3-2-1.** Tres copias de los datos, en dos medios distintos, uno de los cuales debe estar **offsite y offline** —lo que llamamos air-gapped, es decir, physically disconnected. Un ransomware que cifra tu red no puede cifrar una cinta de backup que no está conectada. Además, los backups deben **probarse periódicamente**: un backup que nadie ha probado restaurar es, estadísticamente, un backup que no funciona cuando más se necesita.

**EDR y XDR.** Los antivirus tradicionales basados en firmas son insuficientes para detectar ransomware polimórfico o variantes desconocidas. Las soluciones de **Endpoint Detection & Response** como CrowdStrike Falcon, SentinelOne o Microsoft Defender for Endpoint utilizan análisis de comportamiento, machine learning e integración con threat intelligence para detectar indicadores de ataque —IOAs— incluso antes de que el payload se ejecute.

**MFA obligatorio en todos los accesos remotos.** El 27% de los ataques entra por RDP con credenciales válidas. MFA en RDP, VPN, y cuentas de administrador de dominio cierra esa puerta de manera drástica.

**Hardening y gestión de superficies de ataque.** Deshabilitar macros de Office por defecto, filtrar tráfico de salida hacia dominios de C2 conocidos, y limitar procesos ejecutables mediante **Application Allowlisting**.

**Control de subida de archivos y ejecución en servidores web.** En los laboratorios de DVWA practicamos exactamente este vector: si un servidor acepta cualquier extensión en un formulario de subida, un atacante carga un `.php` malicioso, lo accede por URL y tiene una **webshell** —ejecución remota de comandos sobre el servidor, incluyendo reverse shell—. Lo que usamos con msfvenom y Netcat en el laboratorio es exactamente lo que hacen los grupos de ransomware para tomar control de servidores web expuestos. La defensa correcta: **lista blanca de extensiones** aceptadas, renombrar el archivo con hash aleatorio al guardarlo, y almacenarlo fuera del directorio público del web server.

**Validación estricta de entradas en backend.** Relacionado con el punto anterior: si una aplicación pasa entradas del usuario directamente al sistema operativo sin validar —Command Injection—, un atacante encadena comandos con `;` y lee el `/etc/passwd` o gana una shell completa. Al mismo tiempo, entradas no sanitizadas en campos HTML permiten **XSS reflejado o almacenado**: en clase vimos cómo un script `<script>new Image().src=...+document.cookie</script>` enviaba la sesión de cualquier usuario al atacante. La sanitización con `htmlspecialchars()` o `DOMPurify` en JavaScript convierte el `<` en `&lt;` y el navegador no lo ejecuta como código. Y el `maxlength` de un campo HTML **no es un control de seguridad**: con F12 se altera en segundos; la validación real está en el backend.

**WAF — Web Application Firewall.** Detecta y bloquea payloads de XSS, SQL Injection y CSRF en tráfico HTTP/S. El **CSRF** —Cross-Site Request Forgery— es otro vector que practicamos: el navegador envía automáticamente cookies con cada petición, y si el cambio de contraseña usa método GET, cualquier enlace malicioso puede ejecutarlo en nombre de la víctima autenticada. Defensa: tokens CSRF por petición y atributo `SameSite` en cookies.

---

### Diapositiva: Contención — Respuesta ante Incidente Activo

**DANIEL:**
Ahora el escenario que nadie quiere pero todos deben tener planificado: el ransomware ya entró, ya se está ejecutando. ¿Qué hacemos?

El primer principio, y lo subrayo porque es contraintuitivo: **NO apagar los equipos comprometidos.** La memoria RAM puede contener claves de cifrado, artefactos del malware o evidencia forense crítica para la investigación. Si se apaga, esa información desaparece.

**Paso 1 — Contención inmediata.** Aislar los equipos afectados de la red: desconectar el cable de red o bloquear el puerto del switch, implementar reglas de aislamiento en el EDR. Pero no apagarlos. Si hay sistemas críticos que no pueden desconectarse —un servidor de producción, por ejemplo— se puede implementar contención a nivel de firewall bloqueando el tráfico lateral.

**Paso 2 — Identificación de la variante y sus CVEs.** Determinar qué variante de ransomware es usando la nota de rescate, la extensión de archivos cifrados o hashes del malware. Herramientas como **ID Ransomware** (id-ransomware.malwarehunterteam.com) o el proyecto **No More Ransom** (nomoreransom.org) —avalado por Europol— pueden identificar la variante y ofrecer decryptors gratuitos si existen. Paralelamente, buscar en la **NVD** y MITRE los CVEs asociados al grupo atacante: esto permite identificar el vector de entrada exacto y aplicar remediación inmediata a todos los sistemas que aún estén expuestos al mismo CVE —porque si entró por ahí en un servidor, puede haber más en la red que aún no fueron comprometidos.

**Paso 3 — Análisis de impacto y alcance.** ¿Cuántos equipos están afectados? ¿Llegó al controlador de dominio? ¿Fueron comprometidos los backups? Esta evaluación define si la respuesta es una recuperación quirúrgica o un rebuild completo.

**Paso 4 — Notificación.** Activar el Plan de Respuesta a Incidentes, notificar a la dirección, y si el ataque involucra datos personales, notificar a la **SIC** —Superintendencia de Industria y Comercio— dentro de los plazos legales que establece la Ley 1581. Si hay afectación a infraestructura crítica, notificar al **ColCERT** y al Centro Cibernético de la Policía Nacional.

**Paso 5 — Erradicación y recuperación.** Reconstruir los sistemas desde imágenes limpias, restaurar desde los backups verificados, y **cambiar TODAS las credenciales** —especialmente las del Directorio Activo, ya que el atacante puede haber dejado persistencia con accesos válidos.

**Paso 6 — Lecciones aprendidas.** El análisis post-incidente no es opcional. ¿Cómo entró? ¿Qué control falló? ¿Cuánto tiempo pasó sin ser detectado? El **dwell time** —tiempo de permanencia del atacante en la red antes de ser descubierto— promedio en 2024 fue de **24 días**. Eso es un mes entero con un atacante mirando todo en tu red.

Y sobre la pregunta del millón: **¿pagar o no pagar?** La posición del FBI, Europol, el CCOC de Colombia y prácticamente todos los organismos de seguridad del mundo es clara: **no pagar**. No hay garantía de recuperar los datos —solo el 24% los recupera completamente—, pagar financia la siguiente operación criminal, y en algunos casos puede violar restricciones de la **lista OFAC del Departamento del Tesoro de EE.UU.** si el grupo receptor está sancionado internacionalmente.

---

### Diapositiva: El Factor Humano — La Capa que más Importa

**DANIEL:**
Todo lo técnico que acabo de describir falla si el factor humano no está cubierto.

El 90% de los incidentes de seguridad exitosos tienen como punto de entrada un error humano. El firewall más sofisticado del mundo no detiene a un empleado que hace clic en un enlace de phishing.

Por eso, la capacitación no es un lujo: es un control de seguridad. **Simulaciones de phishing** periódicas, talleres de concienciación, y una cultura organizacional donde reportar un correo sospechoso no genera vergüenza sino que es reconocido como una acción correcta.

Asimismo, los ejercicios **Tabletop** —donde el equipo de dirección y TI simulan responder a un incidente de ransomware sin que haya uno real— son fundamentales para identificar brechas en el plan de respuesta antes de necesitarlo.

---

## 🎯 CONCLUSIONES GENERALES — DANIEL (cierra los 3 temas)

**DANIEL:**
Para cerrar, quiero que nos llevemos conclusiones claras de los tres temas que presentamos hoy.

**Del tema de Verónica:** El ransomware no es nuevo. Lleva más de 35 años evolucionando, y el salto del código artesanal de Joseph Popp al modelo industrial del RaaS representa quizás la mayor transformación del crimen organizado en la era digital. Hoy, para lanzar un ataque de ransomware no se necesita ser un experto en programación; se necesita dinero para contratar un afiliado y acceso a una red corporativa vulnerable. **La democratización del ransomware es nuestra mayor amenaza estructural.**

**Del tema de Cris:** Técnicamente, el ransomware es implacable cuando encuentra el camino. Pero ese camino siempre tiene un nombre: una vulnerabilidad sin parchear, un puerto expuesto, un usuario que abrió un correo. **Ningún ataque ocurre por magia; ocurre porque hubo una brecha que podría haber sido cerrada.** Las estadísticas —mil millones de dólares en rescates, 22 días de inactividad, 24% de recuperación exitosa— deben ser el argumento más poderoso para convencer a la alta dirección de invertir en ciberseguridad antes, no después.

**De mi tema:** Los grupos activos —RansomHub, LockBit, BlackCat, Cl0p— son organizaciones criminales con estructura empresarial, capacidades técnicas de primer nivel y motivación económica ilimitada. Con **35.000 intentos en Colombia** solo en el último año, esta no es una amenaza distante. Nuestra respuesta debe ser sistemática y hablar el mismo lenguaje que aprendemos en la especialización: **CVEs priorizados con CVSS v4.0 y EPSS; hardening de aplicaciones web frente a XSS, CSRF, Command Injection y File Upload; arquitecturas Zero Trust; backups air-gapped; EDR con detección por comportamiento; y planes de IR ensayados con Tabletop.** La ciberseguridad no es un producto que se compra; es un proceso continuo que se construye, se practica en laboratorio y se actualiza con cada nueva amenaza.

Y el mensaje final que quiero dejar, tanto para esta audiencia académica como para cuando ejerzamos como profesionales: **la pregunta no es si su organización va a ser atacada, sino cuándo. La diferencia entre una organización que sobrevive a un ataque de ransomware y una que no, se decide mucho antes de que el ransomware llegue.**

Muchas gracias. Quedamos abiertos a preguntas, y los invitamos a participar en el Kahoot que preparamos para reforzar los conceptos clave de hoy.

---

## ⏱ TIEMPOS ORIENTATIVOS

| Sección | Expositor | Tiempo |
|---------|-----------|--------|
| Apertura + Índice | Verónica | ~1 min |
| Tema 1: RaaS, Antecedentes y Origen | Verónica | ~6 min |
| Tema 2: Funcionamiento, Vectores y Estadísticas | Cris | ~6 min |
| Tema 3: Grupos activos | Daniel | ~3 min |
| Tema 3: CVSS / EPSS / MITRE ATT&CK | Daniel | ~2 min |
| Tema 3: Prevención técnica | Daniel | ~3 min |
| Tema 3: Contención e IR | Daniel | ~3 min |
| Conclusiones | Daniel | ~2 min |
| **TOTAL** | | **~26 min** |

---

## 📌 GLOSARIO DE TÉRMINOS TÉCNICOS USADOS

| Término | Definición rápida |
|---------|-------------------|
| **CVE** | Common Vulnerabilities and Exposures. Identificador único global de vulnerabilidades, gestionado por MITRE. Formato: CVE-AÑO-NÚMERO. |
| **NVD** | National Vulnerability Database. Base de datos del NIST que publica el análisis técnico (CVSS) de cada CVE. |
| **CVSS v4.0** | Common Vulnerability Scoring System v4.0. Estándar (0–10) de severidad técnica con 4 grupos: Base, Threat, Environmental y Supplemental. |
| **CVSS-B / BT / BE / BS** | Los cuatro grupos de métricas de CVSS v4.0: Base / Threat / Environmental / Supplemental. |
| **AT (Attack Requirements)** | Métrica nueva en CVSS v4.0. Indica si el ataque depende de una configuración específica del sistema víctima que el atacante no controla. |
| **Exploit Maturity (E)** | Métrica Threat de CVSS v4.0: Unreported / Proof-of-Concept / Attacked / Confirmed. Reemplaza las antiguas Temporal Metrics de v3.1. |
| **EPSS** | Exploit Prediction Scoring System. Probabilidad (0–1) de que una vulnerabilidad sea explotada en los próximos 30 días. |
| **MITRE ATT&CK** | Framework de tácticas y técnicas observadas en ataques reales. Referencia universal para defensores y atacantes. |
| **EDR / XDR** | Endpoint/Extended Detection & Response. Solución de seguridad con análisis de comportamiento en endpoints. |
| **EDR-Killer** | Técnica usada por grupos como RansomHub para deshabilitar activamente soluciones EDR antes de ejecutar el ransomware. |
| **Zero Trust** | Arquitectura donde ningún usuario ni dispositivo es confiable por defecto, incluso dentro de la red corporativa. |
| **IOA / IOC** | Indicator of Attack / Indicator of Compromise. Señales técnicas de un ataque en curso o pasado. |
| **Air-gapped** | Sistema físicamente desconectado de la red, inaccesible a ransomware remoto. |
| **TTPs** | Tactics, Techniques, and Procedures. Forma en que los actores de amenaza operan (MITRE ATT&CK). |
| **Kill Chain** | Modelo que describe las fases de un ciberataque desde el reconocimiento hasta el impacto. |
| **Dwell time** | Tiempo que un atacante permanece sin ser detectado dentro de una red comprometida (~24 días en 2024). |
| **RaaS** | Ransomware as a Service. Modelo criminal de suscripción/afiliados para desplegar ransomware. |
| **WAF** | Web Application Firewall. Filtra payloads maliciosos (XSS, SQLi, CSRF) en tráfico HTTP/S. |
| **Webshell** | Archivo PHP/ASP malicioso subido a un servidor web que permite ejecución remota de comandos. Vector de entrada estudiado en labs DVWA. |
| **XSS** | Cross-Site Scripting. Inyección de código JavaScript en páginas web. Reflejado (GET) o Almacenado (POST/BD). Mitigación: `htmlspecialchars()`, `DOMPurify`. |
| **CSRF** | Cross-Site Request Forgery. Petición forjada desde sitio externo que el navegador ejecuta con cookies de la víctima. Mitigación: tokens CSRF + `SameSite`. |
| **Command Injection** | Vulnerabilidad en la que entradas del usuario se pasan sin validar al sistema operativo, permitiendo encadenar comandos. |
| **SOAR** | Security Orchestration, Automation and Response. Plataforma que automatiza la respuesta a incidentes. |
| **RansomHub** | Grupo RaaS más activo en 2025 (+1.000 víctimas). Surgió absorbiendo afiliados de LockBit y BlackCat. Usa técnicas EDR-killer. |
| **ColCERT** | Equipo de Respuesta a Emergencias Cibernéticas de Colombia (MinDefensa). |
| **OFAC** | Office of Foreign Assets Control (EE.UU.). Emite listas de grupos sancionados; pagar rescate a un grupo listado puede ser ilegal. |

---

*Diálogo preparado el 03/03/2026 — Exposición: Introducción a la Ciberseguridad*

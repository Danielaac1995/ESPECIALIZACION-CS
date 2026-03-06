# 🎤 Notas de Exposición — Slides 10, 11 y 12
### Act2 Ransomware v3 — Daniel Aguirre
> ⭐ Los temas marcados con **[REFUERZO]** son los que pediste con más profundidad. El resto cubre todos los puntos de las 3 slides.

---

## 🛡️ SLIDE 10 — Prevención

---

### 🔐 Autenticación Robusta — MFA y FIDO2/Passkeys

> *"La primera línea de defensa es verificar que quien dice ser tú, realmente lo eres."*

**¿Qué es MFA?**
- MFA (Multi-Factor Authentication / Autenticación Multifactor) exige **dos o más pruebas de identidad** para ingresar a un sistema.
- Los factores son: algo que **sabes** (contraseña), algo que **tienes** (celular, llave física), algo que **eres** (huella, cara).
- Un atacante que roba tu contraseña sola no puede entrar si además necesita tu celular o tu huella.

**¿Qué es FIDO2 / Passkeys?**
- Es el estándar más seguro de MFA. En vez de contraseña, usa **criptografía de clave pública** vinculada a tu dispositivo físico.
- No se puede robar por phishing porque la clave nunca viaja por internet: se genera y queda en el dispositivo.
- Es lo que usan hoy Google, Apple y Microsoft para reemplazar contraseñas tradicionales.

**¿Por qué es crítico contra ransomware?**
- El 23% de los ataques de 2025 entran por **credenciales comprometidas** (slide 8). Con MFA, esas credenciales robadas no sirven solas para entrar.

**Frase clave:**
> *"Una contraseña robada sin el segundo factor es inútil para el atacante. MFA es el control de mayor retorno por menor costo."*

---

### 🔒 Principio de Mínimo Privilegio

> *"No se trata de desconfiar de las personas, se trata de limitar el daño si algo sale mal."*

**¿Qué es?**
- Cada usuario, sistema o proceso tiene acceso **solo a lo estrictamente necesario** para su función, nada más.
- Un empleado de contabilidad no necesita acceso a los servidores de producción. Un servidor web no necesita acceso a la base de datos de RRHH.

**¿Por qué frena el ransomware?**
- En la fase de **Reconocimiento Lateral** (slide 7), el ransomware intenta moverse de un equipo a otro buscando el controlador de dominio y los backups.
- Si cada cuenta tiene permisos mínimos, el malware no puede saltar libremente: cada movimiento choca con una barrera de permisos.

**Ejemplo práctico:**
> *"Si el ransomware compromete la cuenta de un proveedor externo con acceso mínimo, solo puede cifrar lo que esa cuenta puede ver. Con privilegios amplios, cifraría toda la red."*

---

### 🔄 Gestión de Parches — ¿Qué es un parche y por qué < 48h? **[REFUERZO]**

> *"En la diapositiva anterior vimos que el 32% de los ataques de ransomware en 2025 entran por vulnerabilidades conocidas —es el vector número uno. Eso significa que la puerta ya tiene un hueco, y el atacante lo sabe antes que nosotros."*

**¿Qué es un parche?**
- Un parche (patch) es una **actualización de software** que corrige un error, fallo de seguridad o vulnerabilidad en un programa o sistema operativo.
- Es básicamente el fabricante diciendo: *"encontramos un hueco en nuestro código, aquí está la solución"*.
- Sin el parche, ese hueco queda abierto indefinidamente para que cualquier atacante lo use.

**¿Qué es una vulnerabilidad?**
- Es un fallo en el código de un software que permite a un atacante hacer algo que no debería poder: ejecutar código malicioso, escalar privilegios o acceder a datos sin autorización.
- Cada vulnerabilidad tiene un identificador único llamado **CVE** (Common Vulnerabilities and Exposures). Ejemplo: CVE-2023-20269 es la que usó Akira en VPNs Cisco (slide 12).

**¿Qué es CVSS?**
- Sistema de puntuación del 0 al 10 que mide qué tan grave es una vulnerabilidad.
- 9.0–10.0 = Crítica → parchear en menos de 48h.
- 7.0–8.9 = Alta → parchear en menos de 7 días.

**¿Por qué exactamente 48 horas?**
- Una vulnerabilidad publicada en el catálogo **CISA KEV** (Known Exploited Vulnerabilities) comienza a ser explotada masivamente en promedio **dentro de las primeras 24–72 horas** de su divulgación pública.
- El tiempo promedio entre intrusión y ejecución del ransomware es **9 días** (Sophos 2025). Si parcheas en menos de 48h, cierras la ventana antes de que el atacante la cruce.
- Grupos como **Akira** explotaron el CVE-2023-20269 en VPNs Cisco. **Cl0p** usó 0-days en MOVEit y Cleo. El parche que no aplicas hoy es el vector del ataque de mañana.

**Frase clave:**
> *"Un parche es la respuesta del fabricante a un hueco que ya existe. Mientras no lo apliques, ese hueco está abierto para cualquiera. 48 horas no es un capricho, es el tiempo real que tienes antes de que alguien lo explote."*

---

### 💾 Copias de Respaldo — ¿Qué es un backup y qué significa 3-2-1-1-0? **[REFUERZO]**

> *"El backup es el último recurso cuando todo lo demás falla. Pero un backup mal hecho es como no tener backup."*

**¿Qué es un backup?**
- Una copia de seguridad es una **reproducción de los datos en un momento específico en el tiempo**, guardada en un lugar separado del original.
- Si el ransomware cifra los datos originales, el backup limpio permite restaurarlos sin pagar rescate.
- No es suficiente tener una copia: hay que tener la copia **correcta**, en el **lugar correcto** y **verificada**.

**Explicación número por número de la regla 3-2-1-1-0:**

| Número | Significa | Por qué importa |
|--------|-----------|-----------------|
| **3** | Tres copias del dato | Si una falla o es cifrada, hay dos más |
| **2** | En dos tipos de medios distintos | Ej: disco local + nube. Evita que un solo fallo de hardware lo destruya todo |
| **1** | Una copia fuera del sitio (offsite) | Si el ransomware cifra todo en la oficina, la copia remota queda intacta |
| **1** | Una copia air-gapped | Completamente desconectada de la red. El ransomware **no puede llegar ahí** porque no hay conexión |
| **0** | Cero errores verificados | De nada sirve tener copias si al restaurar están corruptas. Se deben probar periódicamente |

**Dato clave:**
> *"Sophos reporta que el 97% de las organizaciones con backups sanos logró recuperarse sin pagar el rescate. El 0 de la regla es el más importante: un backup que nunca se prueba, no existe."*

---

### 🌐 Segmentación de Red y Zero Trust **[REFUERZO]**

> *"Imaginemos que el ransomware ya entró. ¿Cómo limitamos el daño?"*

**¿Qué es la segmentación de red?**
- Dividir la red corporativa en **zonas aisladas** (microsegmentación) que no se comunican libremente entre sí.
- Ejemplo: la red de finanzas separada de producción, separada de usuarios, separada de backups.
- Si el ransomware entra por el equipo de un empleado, **no puede saltar automáticamente** al servidor de backups o al controlador de dominio porque hay una barrera entre ellos.
- En la slide 7 vemos que en la fase de **Reconocimiento Lateral** el atacante mapea la red buscando esos servidores críticos. La segmentación hace ese mapeo mucho más difícil y lento.

**¿Qué es Zero Trust?**
- Es un modelo de seguridad basado en el principio: **"nunca confiar, siempre verificar"**.
- En el modelo tradicional: si estás dentro de la red corporativa, eres de confianza automáticamente. Zero Trust elimina esa confianza implícita.

**Los 3 principios de Zero Trust:**
1. **Nunca confiar, siempre verificar** → cada acción se autentica, sin importar si el usuario ya está dentro de la red.
2. **Mínimo privilegio** → nadie tiene más acceso del necesario (conecta con la tarjeta anterior).
3. **Asumir compromiso** → operar como si ya hubiera un intruso dentro. Eso obliga a monitorear constantemente.

**Frase clave:**
> *"El modelo tradicional decía: si estás dentro de la red, eres de confianza. Zero Trust dice lo contrario: estar dentro no te da nada. Demuéstralo cada vez."*

---

### 📚 Cultura y Formación Continua

> *"Los controles técnicos no sirven si el eslabón humano falla."*

**¿Por qué es un control de seguridad?**
- El 18% de los ataques de 2025 entran por **phishing** (slide 8), y ese porcentaje sube porque la IA generativa hace los correos casi indetectables visualmente.
- Ningún firewall, EDR ni segmentación detiene a un empleado que hace clic en un enlace malicioso y entrega sus credenciales voluntariamente.

**¿Qué incluye la formación continua?**
- **Simulacros de phishing**: enviar correos falsos internamente para medir cuántos empleados caen, y capacitar a los que fallan sin represalias.
- **Capacitación periódica**: no una vez al año, sino continua. Las técnicas de ataque cambian y el personal debe actualizarse al mismo ritmo.
- **Política de reporte**: que cualquier empleado sepa a quién y cómo reportar algo sospechoso. Un reporte a tiempo puede detener un ataque completo.

**Frase clave:**
> *"Un clic equivocado puede activar todo el ataque. La formación continua es el control más económico y el más ignorado."*

---

## 🚨 SLIDE 11 — Contención

---

### ⚡ Aislamiento Inmediato

> *"Desconectar antes de entender. Cada segundo conectado es un sistema más cifrado."*

**¿Qué significa aislar un sistema?**
- Desconectar física o lógicamente el sistema comprometido de la red: apagar la conexión de red, deshabilitar el puerto del switch, o usar el EDR para aislar el endpoint remotamente desde consola.
- **Importante: no apagar el equipo**. Al apagarlo se pierde la memoria RAM, que contiene datos volátiles clave para el análisis forense: claves de cifrado activas, procesos del malware, conexiones abiertas.
- El objetivo es **cortar la propagación lateral** sin destruir evidencia.

**¿Qué pasa si no se aísla rápido?**
- En la slide 7 vimos que el ransomware ya pasó días en la red haciendo reconocimiento. Al activarse, puede cifrar decenas de sistemas en minutos.
- Cada minuto de retraso puede significar cientos de equipos adicionales cifrados y más datos exfiltrados.

**Frase clave:**
> *"Primero cortar, luego entender. El orden importa más que la velocidad de análisis."*

---

### 🔍 EDR y XDR — ¿Qué son? **[REFUERZO]**

> *"Para contener un incidente necesitamos primero verlo. Ahí entran EDR y XDR."*

**EDR — Endpoint Detection and Response:**
- Software instalado en cada **endpoint** (PC, servidor, portátil).
- Monitorea en tiempo real comportamientos anómalos: cifrado masivo de archivos, eliminación de shadow copies, uso de herramientas LOLBins como PsExec o AnyDesk (que vimos en slide 12 con Akira).
- Cuando detecta algo sospechoso, puede **aislar automáticamente** ese equipo de la red sin intervención humana.

**XDR — Extended Detection and Response:**
- Es EDR pero extendido a **toda la infraestructura**: red, nube, correo, identidad.
- Correlaciona señales de múltiples fuentes para detectar ataques que individualmente parecen normales pero juntos son una intrusión.
- Ejemplo: un usuario que se conecta a las 3am, descarga 50GB y luego ejecuta un script de PowerShell → individualmente son eventos normales, juntos son una alerta crítica.

**Diferencia clave:**

| | EDR | XDR |
|---|-----|-----|
| Alcance | Un endpoint | Toda la infraestructura |
| Visibilidad | Un equipo a la vez | Correlación cruzada de señales |
| Ideal para | Detectar en el dispositivo | Detectar ataques distribuidos |

**Conexión con el resto:**
> *"En slide 7 vimos que el atacante pasa en promedio 9 días en la red antes de ejecutar. EDR/XDR es lo que puede detectarlo en esos 9 días, antes de que cifre."*

---

### 🚨 Activación del Equipo — CSIRT y COLCERT **[REFUERZO]**

> *"Un incidente de ransomware no lo maneja una sola persona. Hay una cadena de respuesta definida."*

**CSIRT — Computer Security Incident Response Team:**
- Equipo interno o contratado especializado en respuesta a incidentes de seguridad.
- Su trabajo: **coordinar la respuesta técnica** → aislar, analizar, preservar evidencia, restaurar.
- Lo clave: los roles deben estar definidos **antes** del incidente. Durante el fuego no se improvisa quién hace qué.

**COLCERT — Colombia:**
- Es el **Grupo de Respuesta a Emergencias Cibernéticas de Colombia**, coordinado por el MinTIC.
- Obligatorio notificarles cuando el incidente afecta **infraestructura crítica o datos sensibles**.
- Ellos emiten las alertas que ya vimos en slide 9: Medusa, Phobos, Akira, Ymir activos en Colombia. Esa inteligencia viene de los reportes que las organizaciones hacen a COLCERT.

**Por qué notificar rápido:**
> *"COLCERT puede compartir indicadores de compromiso con otras organizaciones del país para que se protejan antes de ser golpeadas. Reportar no es admitir derrota, es parte de la defensa colectiva."*

---

### 🔎 Preservación de Evidencia

> *"Antes de limpiar, capturar. Sin evidencia no hay causa raíz, y sin causa raíz el ataque se repite."*

**¿Qué es y por qué importa?**
- Antes de restaurar o limpiar cualquier sistema, se debe **capturar una imagen forense**: copia exacta del disco duro y volcado de la memoria RAM.
- La memoria RAM contiene datos volátiles que desaparecen al apagar el equipo: claves de cifrado activas, procesos en ejecución del malware, conexiones de red abiertas al servidor C&C.
- Los logs del sistema registran qué pasó, cuándo y cómo. Son la línea de tiempo completa del ataque.

**¿Qué se busca en el análisis forense?**
- **Vector de entrada**: ¿por dónde entró? (correo, VPN, RDP).
- **Movimiento lateral**: ¿a qué otros sistemas llegó y cuándo?
- **Datos exfiltrados**: ¿qué información salió antes del cifrado?
- **Variante del ransomware**: ¿qué grupo fue? ¿hay decryptor público disponible?

**Frase clave:**
> *"Sin evidencia forense el informe dice 'fue ransomware'. Con evidencia dice 'entró por esta cuenta, se movió a estos sistemas, exfiltró estos archivos'. Esa diferencia es lo que permite mejorar y prevenir el siguiente."*

---

### ♻️ Restauración desde Backup Limpio

> *"No todo backup es válido para restaurar después de un ransomware."*

**¿Qué significa "limpio"?**
- La copia debe ser **anterior al momento de la intrusión**, no solo al cifrado.
- El atacante estuvo en promedio **9 días** en la red antes de cifrar (slide 7). Un backup hecho 3 días antes del cifrado puede ya contener el malware durmiente instalado y persistente.
- Por eso se analiza el backup antes de restaurar: confirmar que no tiene rastros del malware.

**El proceso correcto:**
1. Identificar la fecha aproximada de intrusión con los logs del análisis forense.
2. Seleccionar el backup **anterior a esa fecha**.
3. Restaurar primero en un entorno aislado para verificar integridad y que no hay malware.
4. Solo entonces restaurar en producción.

**Frase clave:**
> *"Restaurar mal es reintroducir el problema. El backup más reciente puede estar ya comprometido."*

---

### 📋 Lecciones Aprendidas

> *"El incidente que no se documenta, se repite."*

**¿Qué es y para qué sirve?**
- Una vez controlado el incidente, se realiza un análisis completo: causa raíz, cronología detallada, impacto real, qué controles funcionaron y cuáles fallaron.
- Ese documento se integra al **plan de gestión de riesgos** de la organización para mejorar los controles existentes.
- Es el paso que convierte un incidente en una mejora de seguridad: cierra el ciclo de prevención → detección → contención → aprendizaje → mejor prevención.

**Frase clave:**
> *"Cerrar el ciclo aquí es lo que convierte un ataque en una inversión de seguridad. Sin este paso, la organización queda igual de vulnerable que antes."*

---

## 🦠 SLIDE 12 — Grupos más activos

> *"Esta slide aterriza todo lo anterior. Estos son los actores reales que usan exactamente los vectores y técnicas que ya explicamos."*

---

**#1 Qilin**
- Modelo **RaaS puro** (slide 6): desarrolladores que alquilan el ransomware a afiliados a cambio de comisión del rescate.
- Absorbió afiliados de RansomHub cuando este cerró en abril 2025 → el ecosistema criminal no desaparece, se reagrupa (conclusión 01 de slide 13).
- **Doble extorsión**: cifra Y amenaza con publicar los datos robados para presionar el pago.
- Dato impactante: su ataque al NHS (sistema de salud del Reino Unido) estuvo vinculado a la muerte de un paciente por retrasos en cirugías.
> *"1.001 víctimas en un año. El grupo más letal de 2025. Llegó porque LockBit cayó, no porque el ransomware bajó."*

**#2 Akira**
- Explota **CVE-2023-20269** en VPNs Cisco → vulnerabilidad conocida, con parche disponible desde 2023. Conexión directa con slide 10 (parches <48h).
- Usa **AnyDesk y LOLBins** (herramientas legítimas del sistema para moverse sin levantar alertas en antivirus tradicionales) → por eso el EDR comportamental es clave (slide 11).
- Vínculos con el extinto grupo **Conti**: los grupos se disuelven pero los operadores reaparecen con nuevo nombre.
- $244M extorsionados. Objetivo principal: salud y transporte, sectores críticos con alertas activas en Colombia.
> *"El CVE que Akira explota tiene parche desde 2023. Cada organización que no lo aplicó en 48h fue un objetivo potencial."*

**#3 Cl0p**
- Especialista en **0-days**: vulnerabilidades que aún no tienen parche porque el fabricante no las conoce públicamente.
- Campañas masivas episódicas: MOVEit 2023, Cleo 2024. Un solo 0-day les da acceso simultáneo a cientos de organizaciones.
- A veces **no cifran**: solo roban y extorsionan con la filtración → el cifrado ya no es el objetivo (conclusión 02 de slide 13).
> *"Cl0p demuestra que esperar el parche no siempre funciona contra 0-days. Por eso el monitoreo de comportamiento con EDR/XDR es igualmente crítico."*

**#4 DragonForce**
- Heredó la infraestructura de **RansomHub** → muestra la resiliencia del modelo RaaS (slide 6).
- Ofrece a sus afiliados "auditoría de datos robados" para saber exactamente qué información tienen y maximizar la presión de extorsión.
- Partnership con **Scattered Spider**: grupo conocido por ingeniería social avanzada → el factor humano de la slide 10 (Cultura y Formación).

**#5 Play**
- Acceso principal vía **RDP comprometido** (Remote Desktop Protocol con credenciales débiles sin MFA) → conexión directa con slide 10 (autenticación robusta, mínimo privilegio).
- Modelo "silencioso": sin teatralidad, sin mucha prensa. Pero CISA reporta ~900 entidades impactadas a mayo 2025.
- Demuestra que los grupos que no hacen ruido son igual de peligrosos que los que sí.

**#6 LockBit 5.0**
- Desmantelado en la **Operación Cronos** (feb 2024) por Europol, FBI y NCA, pero su líder nunca fue capturado → relanzó LockBit 5.0 en septiembre 2025.
- Es la demostración más clara de la conclusión 01 de slide 13: *"el ecosistema criminal es resiliente, se fragmenta pero no colapsa"*.
- Históricamente el grupo con más víctimas acumuladas de todos los tiempos.

---

---

## 💡 Consejo general para el bloque 10–12

Las tres slides forman un argumento completo:

**"Así se previene → así se contiene → estos son los atacantes reales que lo hacen"**

Si logras hacer esa conexión explícita en voz, la exposición queda muy sólida.

**Frase de cierre para todo el bloque:**
> *"Estas no son estadísticas abstractas. Son grupos con nombres, con operadores humanos, con infraestructura activa y con víctimas en Colombia. Todo lo que vimos en prevención y contención está diseñado para frenar exactamente lo que hacen ellos. La pregunta no es si van a intentar atacar, sino si vamos a estar listos."*

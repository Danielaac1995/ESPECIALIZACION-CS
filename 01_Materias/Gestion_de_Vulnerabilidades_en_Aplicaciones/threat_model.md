## Modelo de Amenazas

| Tipo de Amenaza | Escenario | Impacto Potencial |
|-----------------|-----------|-------------------|
| Suplantación (Spoofing) | Un atacante podría hacerse pasar por un autor legítimo de cursos de Kaggle Learn explotando vulnerabilidades en el paquete learntools, lo que podría derivar en la implementación de código malicioso o acceso no autorizado a datos de usuarios. | Ejecución no intencionada de código, brechas de datos o daño reputacional a Kaggle Learn. |
| Suplantación (Spoofing) | Un atacante podría manipular el sistema de plantillas de los notebooks para inyectar contenido malicioso, haciéndolo aparecer como si proviniese de un autor legítimo del curso. | Los usuarios podrían ejecutar código malicioso sin saberlo, comprometiendo sus cuentas o sistemas. |
| Manipulación (Tampering) | Un atacante podría modificar el código de verificación de ejercicios en el paquete learntools para alterar los materiales del curso o la retroalimentación entregada a los usuarios, introduciendo potencialmente vulnerabilidades de seguridad o información engañosa. | Integridad de los materiales del curso comprometida, lo que podría generar problemas de seguridad o retroalimentación incorrecta al usuario. |
| Manipulación (Tampering) | Un atacante podría explotar vulnerabilidades en la infraestructura central del paquete learntools para alterar las entregas de usuarios o los resultados de ejercicios, permitiendo intentos de trampa o manipulación de datos. | Integridad de las entregas y resultados comprometida, lo que podría socavar la efectividad de los cursos de Kaggle Learn. |
| Repudio (Repudiation) | Un usuario podría negar haber completado un ejercicio o haber alcanzado cierta puntuación, alegando un error del sistema o que su cuenta fue comprometida. | Disputas sobre logros de usuarios, lo que podría dañar la reputación de Kaggle Learn o generar solicitudes de soporte innecesarias. |
| Divulgación de Información (Information Disclosure) | Un atacante podría explotar vulnerabilidades en el paquete learntools o en los notebooks para acceder a información sensible de usuarios, como su progreso, entregas o datos personales. | Acceso no autorizado a datos de usuarios, lo que podría derivar en violaciones de privacidad o brechas de seguridad. |
| Divulgación de Información (Information Disclosure) | Un usuario podría divulgar inadvertidamente información sensible, como sus credenciales de cuenta o datos personales, a través de los notebooks o las entregas de ejercicios. | Divulgación no intencionada de datos de usuarios, lo que podría generar problemas de seguridad o preocupaciones de privacidad. |
| Denegación de Servicio (Denial of Service) | Un atacante podría saturar el sistema con un gran número de entregas de ejercicios o solicitudes de notebooks, causando potencialmente problemas de rendimiento o tiempo de inactividad. | Los cursos de Kaggle Learn se vuelven inaccesibles o no responden, impactando negativamente la experiencia del usuario. |
| Denegación de Servicio (Denial of Service) | Un usuario podría accidental o intencionalmente provocar una condición de denegación de servicio enviando ejercicios o notebooks malformados, perturbando el sistema. | Inestabilidad del sistema o tiempo de inactividad, lo que podría afectar a múltiples usuarios o cursos. |
| Elevación de Privilegios (Elevation of Privilege) | Un atacante podría explotar vulnerabilidades en el paquete learntools o en los notebooks para obtener privilegios elevados, permitiéndole acceder o modificar datos sensibles, o realizar acciones no autorizadas. | Acceso no intencionado a datos sensibles o funcionalidades del sistema, lo que podría derivar en brechas de seguridad o corrupción de datos. |
| Elevación de Privilegios (Elevation of Privilege) | Un usuario podría explotar vulnerabilidades del sistema para acceder a las cuentas o datos de otros usuarios, lo que podría llevar a acciones no autorizadas o brechas de datos. | Cuentas o datos de usuarios comprometidos, lo que podría generar problemas de seguridad o daño reputacional. |


## Sugerencias de Mejora

- Proporcionar más detalles sobre el flujo de autenticación entre componentes, como la gestión de sesiones de usuario y el control de acceso a datos sensibles.
- Considerar agregar información sobre los mecanismos de almacenamiento y transmisión de datos utilizados por el paquete learntools y los notebooks, incluyendo medidas de cifrado o control de acceso implementadas.
- Proveer más información sobre la arquitectura del sistema, incluyendo límites de confianza, segmentación de red o mecanismos de aislamiento que puedan impactar el modelo de amenazas.
- Clarificar los roles y responsabilidades de los distintos componentes del sistema, como el sistema de plantillas, el código de verificación de ejercicios y el manejo de entregas de usuarios, para comprender mejor los vectores de ataque potenciales.
 
---

## Gestión de Amenazas

### 1. Inteligencia de Amenazas (Threat Intelligence)

La inteligencia de amenazas consiste en recopilar, analizar y aplicar información sobre amenazas actuales y emergentes para tomar decisiones de seguridad proactivas.

**Tipos de inteligencia:**

| Nivel | Descripción | Audiencia |
|-------|-------------|-----------|
| **Estratégica** | Tendencias globales, actores de amenaza, motivaciones | Alta dirección, CISO |
| **Operacional** | Campañas activas, TTPs de grupos conocidos | Equipo de seguridad |
| **Táctica** | IOCs concretos: IPs, dominios, hashes, reglas YARA | SOC, analistas |

**Proceso:**
```
Recolección → Procesamiento → Análisis → Diseminación → Retroalimentación
```

**Fuentes clave:**
- **MITRE ATT&CK** — Base de conocimiento de TTPs reales de actores de amenaza
- **OSINT** — Fuentes abiertas: foros, redes sociales, dark web
- **ISACs** — Centros de intercambio de información por sector (financiero, salud, etc.)
- **Feeds comerciales** — VirusTotal, Recorded Future, CrowdStrike
- **COLCERT / CCN-CERT** — Organismos nacionales de ciberseguridad

**Aplicación al modelo de amenazas:**  
Los IOCs y TTPs identificados alimentan directamente el inventario de amenazas y permiten actualizar los escenarios del modelo STRIDE con ataques reales documentados.

---

### 2. Cacería de Amenazas (Threat Hunting)

La cacería de amenazas es la búsqueda **proactiva** de amenazas que ya se encuentran dentro de la red o sistema y que los controles automatizados no han detectado (asume brecha).

**Premisa base:** *"El adversario ya está adentro"* — no se espera una alerta, se sale a buscarlo.

**Ciclo de Threat Hunting:**
```
Hipótesis → Investigación → Identificación de patrones → Respuesta → Mejora de detecciones
```

**Tipos de hipótesis:**

| Tipo | Origen | Ejemplo |
|------|--------|---------|
| Basada en inteligencia | IOC/TTP de CTI | Buscar conexiones a C2 conocido de Akira Ransomware |
| Basada en situación | Conocimiento del entorno | Buscar cuentas con escalada de privilegios reciente |
| Basada en analítica | Anomalías estadísticas | Usuarios con volumen inusual de accesos nocturnos |

**Técnicas comunes:**
- Análisis de logs (SIEM): correlación de eventos sospechosos
- Búsqueda de Living-off-the-Land (LOLBins): uso legítimo de herramientas del SO para fines maliciosos
- Análisis de tráfico de red: conexiones inusuales, beaconing
- Análisis de memoria: procesos maliciosos inyectados

**Herramientas:**
- **SIEM**: Splunk, Microsoft Sentinel, Elastic SIEM
- **EDR/XDR**: CrowdStrike Falcon, SentinelOne, Microsoft Defender for Endpoint
- **OpenSource**: Velociraptor, Sigma (reglas de detección), YARA

---

### 3. Respuesta de Incidentes (Incident Response)

La respuesta de incidentes es el proceso estructurado para **detectar, contener, erradicar y recuperarse** de un incidente de seguridad, minimizando el impacto y preservando evidencia.

**Marco de referencia: NIST SP 800-61**

```
Preparación → Detección y Análisis → Contención → Erradicación → Recuperación → Lecciones Aprendidas
```

**Fases en detalle:**

| Fase | Actividades clave |
|------|-------------------|
| **1. Preparación** | Definir el CSIRT, crear playbooks, establecer canales de comunicación, implementar herramientas (EDR, SIEM) |
| **2. Detección y Análisis** | Identificar el incidente, clasificar la severidad, establecer el alcance inicial |
| **3. Contención** | Contención a corto plazo (aislar sistemas), contención a largo plazo (parches, cambio de credenciales) |
| **4. Erradicación** | Eliminar el malware, cerrar el vector de ataque, limpiar artefactos del atacante |
| **5. Recuperación** | Restaurar sistemas desde backups limpios, monitorear activamente post-restauración |
| **6. Lecciones Aprendidas** | Análisis post-mortem, actualizar playbooks, mejorar detecciones |

**Clasificación de severidad (ejemplo):**

| Nivel | Criterio | Tiempo de respuesta |
|-------|----------|---------------------|
| Crítico | Ransomware activo, exfiltración en curso | < 1 hora |
| Alto | Acceso no autorizado a sistemas críticos | < 4 horas |
| Medio | Malware detectado y contenido | < 24 horas |
| Bajo | Phishing sin clic, escaneos externos | < 72 horas |

**Entidades de apoyo en Colombia:**
- **COLCERT** — Grupo de Respuesta a Emergencias Cibernéticas de Colombia
- **csirtPyme** — Para empresas del sector privado (especialmente pymes)
- **Policía Nacional / CAI Virtual** — Reporte de delitos informáticos

**Relación con el modelo de amenazas:**  
Cada escenario del Modelo de Amenazas (sección 1) debe tener un playbook de respuesta asociado. Los IOCs encontrados en incidentes pasados retroalimentan la Inteligencia de Amenazas, cerrando el ciclo.

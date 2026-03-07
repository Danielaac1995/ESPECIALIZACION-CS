# 📓 Notas de Clase — Análisis y Gestión de Riesgos

**Especialización:** Ciberseguridad  
**Docente:**  
**Lugar / Modalidad:**

---

## 📌 Sesión 1 — Threat Modeling Tool (TMT) de Microsoft

**Fecha:** 06/03/2026

### 🛠️ ¿Qué es TMT (Threat Modeling Tool)?

**Microsoft Threat Modeling Tool (TMT)** es una herramienta gratuita de Microsoft que permite identificar y analizar amenazas de seguridad en sistemas de software durante la fase de diseño, antes de que el sistema sea construido o desplegado.

> *"Es más barato y efectivo encontrar amenazas en el diseño que corregirlas en producción."*

#### Características principales

| Característica | Descripción |
|---------------|-------------|
| **Creador** | Microsoft |
| **Licencia** | Gratuita |
| **Metodología base** | STRIDE |
| **Tipo de análisis** | Análisis de amenazas basado en diagramas de flujo de datos (DFD) |
| **Salida** | Lista de amenazas con nivel de riesgo y mitigaciones recomendadas |

---

### 📖 Conceptos clave

| Término | Definición |
|---------|------------|
| **Threat Modeling** | Proceso estructurado para identificar, clasificar y mitigar amenazas de seguridad en un sistema |
| **DFD** | Diagrama de Flujo de Datos — representa cómo fluye la información entre componentes del sistema |
| **STRIDE** | Metodología de Microsoft para clasificar amenazas (ver abajo) |
| **Trust Boundary** | Límite de confianza — frontera entre dos componentes con diferentes niveles de privilegio o confianza |
| **Mitigación** | Control o contramedida aplicada para reducir o eliminar una amenaza |

---

### 🔐 Metodología STRIDE

STRIDE es el modelo de clasificación de amenazas usado por TMT. Cada letra representa un tipo de amenaza:

| Letra | Amenaza (EN) | Amenaza (ES) | Propiedad vulnerada |
|-------|-------------|-------------|-------------------|
| **S** | Spoofing | Suplantación de identidad | Autenticación |
| **T** | Tampering | Manipulación de datos | Integridad |
| **R** | Repudiation | Repudio | No repudio |
| **I** | Information Disclosure | Divulgación de información | Confidencialidad |
| **D** | Denial of Service | Denegación de servicio | Disponibilidad |
| **E** | Elevation of Privilege | Elevación de privilegios | Autorización |

---

### 🖥️ ¿Cómo funciona TMT?

1. **Crear el diagrama** — Se dibuja el sistema usando elementos del DFD:
   - Proceso (círculo)
   - Almacén de datos (líneas paralelas)
   - Entidad externa (rectángulo)
   - Flujo de datos (flecha)
   - Límite de confianza (línea punteada)

2. **Analizar amenazas** — La herramienta genera automáticamente una lista de amenazas STRIDE para cada elemento del diagrama.

3. **Revisar y clasificar** — El analista revisa cada amenaza y la clasifica como:
   - ✅ Mitigada
   - ⚠️ Necesita investigación
   - ❌ No aplicable

4. **Generar reporte** — TMT exporta un informe con todas las amenazas identificadas y las mitigaciones sugeridas.

---

### � Referencias

- Microsoft Threat Modeling Tool: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool
- Microsoft SDL Threat Modeling: https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling
- Metodología STRIDE: https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats

---

## 📌 Sesión 2 — IA en Threat Modeling, Pirámide del Dolor y Herramientas

**Fecha:** 06/03/2026

---

### 🤖 StrideGPT — IA para Threat Modeling

**URL:** https://stridegpt.streamlit.app/

Herramienta basada en IA (LLM) que automatiza el análisis STRIDE. En lugar de dibujar manualmente el DFD y revisar amenaza por amenaza, StrideGPT genera automáticamente:

- **Árbol de ataque** — Estructura jerárquica que descompone un objetivo de ataque en subobjetivos y pasos concretos que un atacante debe seguir para lograrlo.
- **Mitigaciones** — Contramedidas sugeridas por dominio STRIDE para cada componente del sistema.
- **Casos de uso de seguridad** — Escenarios concretos donde se aplica cada control o donde el sistema puede fallar.

#### ¿Cómo funciona?

1. Se describe el sistema en lenguaje natural o se sube un diagrama.
2. La IA clasifica las amenazas con STRIDE automáticamente.
3. Genera el árbol de ataque, las mitigaciones y los casos de uso.
4. Se puede iterar con preguntas de seguimiento.

#### Para generar la API Key (Groq)

StrideGPT usa el API de **Groq** como backend LLM (modelo rápido y gratuito):

1. Ir a https://console.groq.com
2. Crear cuenta gratuita.
3. En el panel: **API Keys → Create API Key**.
4. Copiar la clave y pegarla en StrideGPT al iniciar.

> Groq es una plataforma de inferencia de LLMs de alta velocidad. Ofrece acceso gratuito a modelos como LLaMA y Mixtral mediante API.

---

### 🌳 Árbol de Ataque (Attack Tree)

Modelo visual que representa cómo un atacante puede lograr un objetivo malicioso. Fue propuesto por **Bruce Schneier (1999)**.

```
          [Objetivo raíz — ej: Robar datos de la BD]
                 /                        \
    [Acceso directo a la BD]     [Exfiltrar via app web]
         /        \                    /         \
[Credenciales  [Inyección   [XSS para    [CSRF para
  válidas]       SQL]        robar token]  forzar req.]
```

- **Nodo raíz:** objetivo final del atacante.
- **Nodos intermedios:** subobjetivos — pasos necesarios.
- **Nodos hoja:** acciones concretas y técnicas específicas.
- **AND / OR:** los nodos se conectan con lógica AND (deben cumplirse todos) u OR (basta uno).

**Uso en AGRC:** permite visualizar todos los caminos posibles de ataque y priorizar dónde invertir en controles.

---

### 🔗 Microsoft SDL — Threat Modeling

**URL:** https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling

El **SDL (Security Development Lifecycle)** de Microsoft integra el threat modeling como una fase obligatoria del ciclo de desarrollo seguro. El proceso SDL propone 4 preguntas base:

| Pregunta | Propósito |
|----------|-----------|
| ¿Qué estamos construyendo? | Entender el sistema y sus componentes |
| ¿Qué puede salir mal? | Identificar amenazas con STRIDE |
| ¿Qué hacemos al respecto? | Definir mitigaciones |
| ¿Hicimos un buen trabajo? | Validar que los controles son suficientes |

El SDL usa la TMT como herramienta principal y conecta directamente con STRIDE para que la identificación de amenazas sea sistemática y no dependa de la experiencia individual del analista.

---

### 🔺 Pirámide del Dolor — Inteligencia de Amenazas

**Autor:** David Bianco (2013)  
**Concepto:** Clasifica los indicadores de compromiso (IoC) según el "dolor" que le causa al atacante cuando el defensor los detecta y bloquea.

La clave: **cuanto más arriba en la pirámide puedas detectar y responder, más daño le causas al atacante** — porque esos indicadores son más difíciles y costosos de cambiar.

```
        /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
       /     TTPs (Tough)         \       ← El atacante debe cambiar CÓMO opera
      /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
     /       Tools (Challenging)   \      ← Debe reescribir o reemplazar sus herramientas
    /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
   /  Network/Host Artifacts (Annoying)\  ← Debe modificar configuraciones del malware
  /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
 /      Domain Names (Simple)          \  ← Registra un nuevo dominio
/‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
/         IP Addresses (Easy)            \ ← Cambia de IP con un clic
/‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
/           Hash Values (Trivial)          \ ← Modifica 1 byte del archivo
```

#### Detalle de cada nivel

| Nivel | Indicador | Dolor al atacante | Descripción |
|-------|-----------|-------------------|-------------|
| 🔴 **Tough** | TTPs | Máximo | Tácticas, Técnicas y Procedimientos (framework MITRE ATT&CK). El atacante debe cambiar radicalmente su forma de operar. |
| 🟠 **Challenging** | Tools | Alto | Herramientas que usa: Cobalt Strike, Mimikatz, Metasploit. Debe recrear o reemplazar la herramienta. |
| 🟡 **Annoying** | Network/Host Artifacts | Moderado | Cadenas específicas en el malware, rutas de archivos, claves de registro, user-agents. Debe recompilar o reconfigurar. |
| 🟡 **Simple** | Domain Names | Bajo-Moderado | Dominios C&C. Puede registrar uno nuevo en minutos pero tiene costo y tiempo. |
| 🟢 **Easy** | IP Addresses | Bajo | Cambia de IP en segundos usando VPS o proxies. |
| 🟢 **Trivial** | Hash Values | Mínimo | Modificar 1 byte del archivo cambia el hash completamente. El más fácil de evadir para el atacante. |

#### ¿Cómo se usa en AGRC?

- **Nivel bajo (Trivial/Easy):** detección reactiva — útil pero fácilmente evadida.
- **Nivel alto (Challenging/Tough):** detección basada en comportamiento (EDR/XDR, MITRE ATT&CK) — mucho más robusta y costosa para el atacante.
- La pirámide guía **dónde invertir en capacidades de detección**: un SOC maduro debe operar en los niveles altos.

---

### 🔑 Gestores de Contraseñas Mencionados

| Herramienta | Tipo | Características |
|-------------|------|----------------|
| **KeePass** | Local, open source | Almacena contraseñas cifradas en un archivo `.kdbx` en el equipo. Sin servidor externo. Gratuito. |
| **Bitwarden** | Cloud + local, open source | Sincronización entre dispositivos. Versión gratuita muy completa. Puede autohospedarse. |

> **Nota de clase:** También se mencionó **Bigworder** en el contexto de gestión de contraseñas / diccionarios de ataque — puede referirse a un wordlist generator usado en pruebas de penetración (fuerza bruta).

---

---

### 📋 Inventario de Amenazas

El **inventario de amenazas** es un catálogo estructurado de todas las amenazas potenciales identificadas para un sistema. Es el insumo principal del proceso de análisis de riesgos — sin saber qué amenazas existen, no se puede evaluar ni gestionar el riesgo.

#### ¿Para qué sirve?

- Tener una visión completa y sistemática de lo que puede salir mal.
- Priorizar controles según la probabilidad e impacto de cada amenaza.
- Evitar que amenazas queden sin analizar por descuido o sesgo del analista.
- Reutilizar el catálogo en futuros proyectos similares.

#### Estructura típica de un inventario de amenazas

| Campo | Descripción |
|-------|-------------|
| **ID** | Identificador único (ej. AME-001) |
| **Categoría STRIDE** | S / T / R / I / D / E |
| **Nombre de la amenaza** | Descripción corta |
| **Componente afectado** | Proceso, almacén, flujo de datos, entidad externa |
| **Descripción** | Qué hace el atacante, cómo explota la amenaza |
| **Probabilidad** | Alta / Media / Baja |
| **Impacto** | Alto / Medio / Bajo |
| **Nivel de riesgo** | Probabilidad × Impacto |
| **Mitigación** | Control propuesto |
| **Estado** | Mitigada / Pendiente / Aceptada |

#### Ejemplo de inventario (sistema web con login)

| ID | STRIDE | Amenaza | Componente | Riesgo | Mitigación |
|----|--------|---------|------------|--------|------------|
| AME-001 | S | Suplantación de usuario con credenciales robadas | Login endpoint | Alto | MFA obligatorio |
| AME-002 | T | Manipulación de tokens JWT en tránsito | Flujo usuario→API | Alto | HTTPS + firma del token |
| AME-003 | I | Exposición de datos sensibles en logs | Base de datos | Medio | Enmascarar datos en logs |
| AME-004 | D | Ataque DDoS al endpoint de autenticación | Servidor web | Medio | Rate limiting + WAF |
| AME-005 | E | Escalada de privilegios por IDOR | API REST | Alto | Validación de autorización en cada request |

#### Relación con otras herramientas vistas en clase

```
Sistema (DFD / TMT)
        ↓
  STRIDE por componente
        ↓
  Inventario de Amenazas  ← también alimentado por Pirámide del Dolor y CTI
        ↓
  Árbol de Ataque (profundiza en cada amenaza)
        ↓
  Mitigaciones + Casos de uso (StrideGPT / TMT)
        ↓
  Gestión del Riesgo (priorización y tratamiento)
```

#### Fuentes para construir el inventario

| Fuente | Qué aporta |
|--------|------------|
| **STRIDE** | Clasificación sistemática por tipo de amenaza |
| **MITRE ATT&CK** | TTPs reales usados por atacantes (nivel alto pirámide) |
| **OWASP Top 10** | Amenazas más críticas en aplicaciones web |
| **CVE / NVD** | Vulnerabilidades conocidas en componentes del sistema |
| **Threat Intelligence** | Amenazas activas en el sector/industria específica |

---

### 🔗 Referencias Sesión 2

- StrideGPT: https://stridegpt.streamlit.app/
- Groq API: https://console.groq.com
- Microsoft SDL Threat Modeling: https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling
- Pirámide del Dolor — David Bianco (2013): https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html
- MITRE ATT&CK (TTPs): https://attack.mitre.org
- KeePass: https://keepass.info

---

## 📅 Sesión 3 — 06/03/2026

### 🎯 DREAD — Metodología de Calificación de Riesgo

**DREAD** es un modelo de calificación cuantitativa de amenazas que complementa STRIDE. Mientras STRIDE **clasifica** el tipo de amenaza, DREAD **puntúa** qué tan grave es cada una, permitiendo **priorizar** cuáles atender primero.

#### Las 5 dimensiones de DREAD

| Dimensión | Sigla | Pregunta clave | Escala |
|-----------|-------|----------------|--------|
| **Damage Potential** — Daño Potencial | D | ¿Qué tan grave sería el daño si se explota? | 1–10 |
| **Reproducibility** — Reproducibilidad | R | ¿Qué tan fácil es reproducir el ataque? | 1–10 |
| **Exploitability** — Explotabilidad | E | ¿Qué tan fácil es ejecutar el ataque? | 1–10 |
| **Affected Users** — Usuarios Afectados | A | ¿Cuántos usuarios se ven afectados? | 1–10 |
| **Discoverability** — Descubribilidad | D | ¿Qué tan fácil es descubrir la vulnerabilidad? | 1–10 |

**Fórmula:**
$$\text{Risk Score} = \frac{D + R + E + A + D}{5}$$

#### Interpretación del puntaje

| Rango | Nivel de Riesgo | Acción |
|-------|----------------|--------|
| 7.5 – 10 | 🔴 **Crítico** | Mitigar de inmediato |
| 5.0 – 7.4 | 🟠 **Alto** | Planificar mitigación a corto plazo |
| 2.5 – 4.9 | 🟡 **Medio** | Monitorear y programar mejoras |
| 0 – 2.4 | 🟢 **Bajo** | Aceptar o mitigar en ciclo normal |

#### DREAD + STRIDE: flujo de trabajo

```
DFD del sistema
      ↓
STRIDE por componente → Inventario de Amenazas
      ↓
DREAD por amenaza → Puntuación de riesgo
      ↓
Priorización → Plan de mitigación
```

#### Ventajas y limitaciones

| Ventajas | Limitaciones |
|----------|--------------|
| Simple y rápido de aplicar | Subjetivo si no hay criterios definidos |
| Genera un ranking claro de amenazas | No reemplaza metodologías más rigurosas (CVSS, OCTAVE) |
| Facilita la comunicación con stakeholders | Los puntajes pueden variar según el analista |
| Compatible con StrideGPT y TMT | — |

> **Nota:** StrideGPT genera automáticamente la tabla DREAD para cada amenaza identificada por STRIDE, junto con el árbol de ataque y las mitigaciones.

---

### 🔗 Referencias Sesión 3

- DREAD Threat Risk Assessment Model — OWASP: https://owasp.org/www-community/DREAD_Risk_Assessment_Model
- StrideGPT (genera DREAD automáticamente): https://stridegpt.streamlit.app/
- MITRE ATT&CK: https://attack.mitre.org

---

*Notas actualizadas el 06/03/2026*

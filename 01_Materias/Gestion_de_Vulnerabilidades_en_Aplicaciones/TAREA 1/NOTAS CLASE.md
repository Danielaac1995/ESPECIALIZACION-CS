# Notas de Clase 3 — 28/02/2026

**Tema:** CVSS v4.0 — Common Vulnerability Scoring System

---

## ¿Qué es CVSS?

Estándar matemático desarrollado por **NIST** y mantenido por **FIRST** para cuantificar la **severidad técnica** de vulnerabilidades. Se usa junto con la base de datos **NVD (National Vulnerability Database)**.

La versión **4.0** (lanzada en noviembre 2023) es la más reciente y reemplaza a v3.1.

Produce un score entre **0.0 y 10.0**:

| Score | Severidad |
|---|---|
| 0.0 | None |
| 0.1 – 3.9 | Low |
| 4.0 – 6.9 | Medium |
| 7.0 – 8.9 | High |
| 9.0 – 10.0 | Critical |

---

## Estructura del CVSS v4.0

v4.0 introduce **4 grupos** de métricas (en v3.1 eran 3):

| Grupo | Nomenclatura | Descripción |
|---|---|---|
| **Base Metrics** | CVSS-B | Características intrínsecas de la vulnerabilidad |
| **Threat Metrics** | CVSS-BT | Estado actual del exploit en la wild (antes: Temporal) |
| **Environmental Metrics** | CVSS-BE | Contexto específico de la organización |
| **Supplemental Metrics** | CVSS-BS | Información adicional de contexto (no cambia el score) |

> El score principal es siempre el **CVSS-B (Base)**. Los demás lo refinan.

---

## Métricas Base (Exploitability)

### Attack Vector (AV) — ¿Desde dónde se explota?

| Valor | Significado |
|---|---|
| N | Network — explotable remotamente por internet |
| A | Adjacent — requiere acceso a la red local |
| L | Local — requiere sesión en el sistema |
| P | Physical — requiere acceso físico al equipo |

### Attack Complexity (AC) — ¿Qué tan difícil es reproducir?

| Valor | Significado |
|---|---|
| L | Low — el ataque es reproducible en cualquier momento |
| H | High — requiere condiciones especiales no controladas por el atacante |

### Attack Requirements (AT) — ⭐ NUEVO en v4.0

Condiciones de despliegue del sistema víctima que el atacante **no controla**:

| Valor | Significado |
|---|---|
| N | None — no hay prerequisitos de configuración |
| P | Present — el ataque depende de una configuración específica del sistema |

> En v3.1 esto estaba parcialmente absorbido por AC. Ahora son métricas separadas.

### Privileges Required (PR) — ¿Qué privilegios necesita el atacante?

| Valor | Significado |
|---|---|
| N | None — **más crítico** |
| L | Low |
| H | High |

### User Interaction (UI) — ⭐ Cambia en v4.0

En v3.1 era `N/R`. En v4.0 tiene 3 valores:

| Valor | Significado |
|---|---|
| N | None — sin interacción |
| P | Passive — la víctima simplemente navega/accede (ej. ver una página) |
| A | Active — la víctima debe realizar una acción deliberada (ej. clic, descarga) |

---

## Métricas de Impacto — ⭐ Gran cambio respecto a v3.1

En v3.1 había **Scope (S)** + CIA una sola vez.
En v4.0 **se eliminó Scope** y se divide el impacto en **dos sistemas**:

| Sistema | Métricas | Descripción |
|---|---|---|
| **Vulnerable System** | VC / VI / VA | Impacto directo en el componente explotado |
| **Subsequent System** | SC / SI / SA | Impacto en otros sistemas fuera del componente (antes: Scope Changed) |

Valores para cada una: `H` (High) / `L` (Low) / `N` (None)

| Código | Nombre | High = |
|---|---|---|
| VC | Vulnerable System Confidentiality | Robo total de datos del sistema explotado |
| VI | Vulnerable System Integrity | Modificación total del sistema explotado |
| VA | Vulnerable System Availability | Sistema explotado inutilizable |
| SC | Subsequent System Confidentiality | Impacto de confidencialidad en sistemas adyacentes |
| SI | Subsequent System Integrity | Modificación de sistemas adyacentes |
| SA | Subsequent System Availability | Caída de sistemas adyacentes |

---

## Threat Metrics (antes: Temporal)

En v4.0 se simplifica a **una sola métrica**:

### Exploit Maturity (E)

| Valor | Significado |
|---|---|
| Unreported | Sin evidencia de explotación |
| Proof-of-Concept | Existe PoC público pero no se usa en ataques reales |
| Attacked | Se usa activamente en ataques |
| Confirmed | Explotación masiva confirmada |

---

## Supplemental Metrics — ⭐ NUEVO en v4.0

No modifican el score numérico, pero aportan contexto:

| Métrica | Valores | Descripción |
|---|---|---|
| Safety (S) | Negligible / Present | ¿Puede afectar la seguridad física de personas? |
| Automatable (AU) | No / Yes | ¿Puede automatizarse el ataque a escala? |
| Recovery (R) | Automatic / User / Irrecoverable | ¿Se puede recuperar el sistema? |
| Value Density (V) | Diffuse / Concentrated | ¿Cuántos recursos controla el sistema explotado? |
| Vulnerability Response Effort (RE) | Low / Moderate / High | Esfuerzo para mitigar |
| Provider Urgency (U) | Clear / Green / Amber / Red | Urgencia declarada por el proveedor |

---

## Ejemplo de Vector CVSS v4.0

```
CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N
```

| Métrica | Valor | Interpretación |
|---|---|---|
| AV | N | Ataque remoto |
| AC | L | Baja complejidad |
| AT | N | Sin requisitos especiales del sistema |
| PR | N | Sin privilegios |
| UI | N | Sin interacción del usuario |
| VC/VI/VA | H/H/H | Impacto total en el sistema vulnerable |
| SC/SI/SA | N/N/N | Sin impacto en sistemas adyacentes |

**Score resultante: 10.0 — Critical**

---

## v3.1 vs v4.0 — Diferencias clave

| Aspecto | v3.1 | v4.0 |
|---|---|---|
| Grupos de métricas | 3 | 4 (+ Supplemental) |
| Temporal → | Temporal Metrics | **Threat Metrics** |
| Scope | Métrica S (U/C) | **Eliminado** — reemplazado por VC/VI/VA + SC/SI/SA |
| Attack Complexity | Incluía prerequisitos | Solo reproducibilidad |
| Attack Requirements | No existía | **Nueva métrica AT** |
| User Interaction | N / Required | N / Passive / **Active** |
| Exploit Maturity | 5 valores | 4 valores (simplificado) |

---

## Cálculo del Base Score (conceptual)

Base Score = f(Exploitability + Impact del Vulnerable System + Impact del Subsequent System)

- **Exploitability** depende de: `AV, AC, AT, PR, UI`
- **Impact** depende de: `VC, VI, VA, SC, SI, SA`

> Función no lineal — usa lookup tables internas definidas por FIRST.

---

## CVSS vs Riesgo Real

**Riesgo = Severidad × Probabilidad × Impacto de negocio**

**CVSS solo mide severidad técnica.** No mide:
- Riesgo organizacional directo
- Probabilidad real de explotación
- Contexto empresarial
- Si hay exploit activo en la wild (eso corresponde a las **Threat Metrics**)



🔎 ¿Qué es un CVE?

CVE significa:

Common Vulnerabilities and Exposures

Es un identificador único público asignado a una vulnerabilidad de seguridad conocida.

El programa CVE es gestionado por la organización MITRE Corporation y utilizado por entidades como el National Institute of Standards and Technology a través de la National Vulnerability Database.

📌 ¿Qué problema resuelve?

Antes del CVE, la misma vulnerabilidad podía tener:

Nombres distintos

Referencias distintas

Descripciones distintas

Ejemplo:
Un vendor la llamaba “Bug X”
Otro la llamaba “Exploit Y”

CVE crea un ID estándar global.

📌 Formato de un CVE
CVE-AÑO-NÚMERO

Ejemplo:

CVE-2021-44228

Ese corresponde a la vulnerabilidad de Log4j.

📌 ¿Qué contiene un CVE?

Un CVE NO es el análisis completo.

Un CVE solo contiene:

Identificador único

Descripción breve

Referencias

Estado (reservado, publicado, rechazado)

El análisis técnico (CVSS, impacto, etc.) lo agrega la National Vulnerability Database u otras bases.

📌 Diferencia clave: CVE vs CVSS
CVE	CVSS
Identificador	Sistema de scoring
Describe la vulnerabilidad	Mide severidad
Es un ID	Es una fórmula matemática

Ejemplo:

CVE-2021-44228
CVSS 3.1 Score: 10.0

CVE identifica.
CVSS mide.


10.2.3.89

GET xss script ( envia variables por la URL)
xss stored POST 

AHORA RETOOO
FILE INCLUSION 
mostrar los usuarios del sistema operativo


PISTAS PARA ESTE RETO

http://localhost:8080/login.php


Para apagarlo: sudo docker stop dvwa
Para encenderlo de nuevo: sudo docker start dvwa

RETO 9 
mediante la vulnerabilidad de command execution imprimir los usuarios del OS 

RETO 10
mediante la vulnerabilidad de command execution ganar una shell del OS de metasploitablle

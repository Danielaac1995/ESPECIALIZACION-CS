# Propuesta de Tesis

**Especialización:** Ciberseguridad  
**Institución:** Instituto Tecnológico Metropolitano (ITM)  
**Fecha:** 05/03/2026  
**Estudiante:Daniel Alejandro Aguirre Ceballos**  
**Director propuesto:**  

---

## 1. Título

> *Modelo de ciberseguridad aplicado a la infraestructura como servicio IaaS usada en la nube híbrida para pymes, con base en gestión de riesgos*

---

## 2. Identificación del Planteamiento del Problema

**Tesis de referencia:** Modelo de ciberseguridad aplicado a la infraestructura como servicio IaaS usada en la nube híbrida para pymes, con base en gestión de riesgos  
**Autor:** Andrey Fabián Moncada García | **ITM, 2025**  
**URL:** https://repositorio.itm.edu.co/entities/publication/f63077de-e4b6-4938-9274-e488fafec12a

---

### ¿Qué es una PYME?

Una **PYME (Pequeña y Mediana Empresa)** es una empresa con recursos limitados en comparación con las grandes corporaciones. En Colombia se clasifican principalmente por número de empleados:

| Tipo | Empleados |
|------|-----------|
| Microempresa | Menos de 10 |
| Pequeña empresa | 10 – 50 |
| Mediana empresa | 51 – 200 |

Las pymes representan el motor de la economía colombiana. Sin embargo, tienen **presupuestos ajustados**, **equipos de TI reducidos** y **poca experiencia en ciberseguridad**, lo que las hace especialmente vulnerables y, a la vez, las deja fuera del alcance de los modelos de seguridad costosos diseñados para grandes empresas.

---

### Descripción del problema

Las pymes están migrando a servicios de nube híbrida (IaaS) por los beneficios de costo y flexibilidad, pero esta migración las expone a riesgos de ciberseguridad para los cuales **no tienen modelos adecuados ni accesibles**.

### Evidencia del problema

- **200.000 millones** de intentos de ciberataques en América Latina en 2023 — Colombia es el **3er país más afectado** (Fortinet, 2023).
- El **58%** de las empresas colombianas no realiza evaluación de riesgos por **falta de modelos bien definidos** (ACIS, 2021).
- El **47,98%** de las empresas encuestadas son pymes (< 500 empleados).
- Los modelos de ciberseguridad existentes son **individuales, costosos o atados a un solo proveedor de nube**, y no se adaptan al contexto de una pyme.

### En palabras del autor de referencia

> *"Se carece de modelos de ciberseguridad bien documentados… los modelos existentes son individuales o para una plataforma específica y no se ajustan a la necesidad de una pyme."*

---

## 3. Justificación

Las pymes representan un gran porcentaje de la economía colombiana y están adoptando aceleradamente servicios IaaS en nube híbrida. Sin embargo, carecen de un modelo de ciberseguridad práctico, escalable y económico que se ajuste a sus capacidades reales. Los estándares existentes (ISO 27001, NIST) son robustos pero complejos y costosos de implementar sin una guía adaptada. Esto genera una brecha crítica de seguridad que expone a estas empresas a amenazas cibernéticas crecientes.

---

## 4. Objetivo General

> Proponer un modelo de ciberseguridad para IaaS en nube híbrida que permita la **reducción de niveles de exposición a riesgos** en empresas pymes, mediante normas internacionales.

---

## 5. Objetivos Específicos

1. Caracterizar los servicios IaaS que puede tener una empresa pyme para su nube híbrida.
2. Analizar estándares, normas y buenas prácticas de seguridad aplicables a servicios en la nube híbrida.
3. Clasificar los principales riesgos en la nube híbrida con base en la norma **ISO 27005**.
4. Aplicar el modelo mediante una simulación o estudio de caso real.

---

## 6. Resultado Final de la Tesis

> **Tipo de resultado:** Modelo de ciberseguridad (no es una arquitectura, ni una estrategia aislada — es un **modelo estructurado con dominios, controles y metodología de evaluación**)

El resultado final es un **Modelo de Ciberseguridad para IaaS en Nube Híbrida** orientado a pymes, compuesto por:

### 1. Arquitectura de referencia para pymes en nube híbrida
Una arquitectura tipo que caracteriza los servicios IaaS que una pyme puede tener (cómputo, almacenamiento, red, seguridad perimetral) sobre la cual se aplica el modelo.

### 2. Modelo estructurado en 7 dominios de seguridad
Los controles de ISO 27001, NIST SP 800-53, CSA, COBIT e ITIL fueron unificados y agrupados en 7 dominios:

| # | Dominio | Qué controla |
|---|---------|-------------|
| 1 | **Gestión de accesos** | Autenticación, IAM, MFA, roles y privilegios |
| 2 | **Gestión de riesgos** | Identificación, análisis y mitigación de amenazas |
| 3 | **Protección de datos** | Cifrado, DLP, clasificación, retención de datos |
| 4 | **Gestión de incidentes** | Detección, respuesta, recuperación y forense |
| 5 | **Monitoreo de actividades** | Logs, SIEM, correlación de eventos |
| 6 | **Gestión de configuraciones** | Hardening, cambios, estandarización |
| 7 | **Continuidad y recuperación** | Backups, DRP, pruebas de restauración |

### 3. Metodología de valoración de riesgos (ISO 27005)
Matriz de riesgos que evalúa **impacto en tiempo** e **impacto en alcance** vs. probabilidad, clasificando cada riesgo como: 🟢 Aceptable / 🟡 Tolerable / 🔴 Inaceptable.

### 4. Formularios de cumplimiento por dominio
Checklists de controles con tres estados de cumplimiento:
- **CC** — Cumple Completamente
- **CP** — Cumple Parcialmente
- **NA** — No Aplica

### 5. Validación en caso de estudio real
El modelo fue aplicado sobre una pyme real con infraestructura IaaS en nube híbrida, evidenciando su **efectividad, adaptabilidad** y generando recomendaciones específicas de mejora.

> **En palabras del autor:**
> *"El modelo es viable y adaptable para las empresas tipo pymes… en una empresa que tenga menos recursos tanto económicos como a nivel de documentación óptima, el modelo entregado le permitirá hacer un análisis y definir los controles que debe tener en cuenta para asegurar su infraestructura."*

---

## 7. Metodología (preliminar)

| Fase | Actividad |
|------|-----------|
| 1 | Caracterización de servicios IaaS en pymes colombianas |
| 2 | Revisión y análisis de estándares internacionales (ISO 27001, NIST SP 800-53, CSA) |
| 3 | Clasificación y valoración de riesgos (ISO 27005) |
| 4 | Diseño del modelo de ciberseguridad |
| 5 | Validación mediante caso de estudio real |

---

## 7. Resultados esperados

- Modelo de ciberseguridad adaptado y documentado para pymes en entornos IaaS / nube híbrida.
- Guía de implementación accesible sin grandes inversiones ni equipos especializados.
- Matriz de riesgos valorada y controles asociados para los principales vectores de amenaza.

---

## 8. Referencias base

- Moncada García, A. F. (2025). *Modelo de ciberseguridad aplicado a la infraestructura como servicio IaaS usada en la nube híbrida para pymes, con base en gestión de riesgos.* [Tesis de Maestría, ITM]. https://repositorio.itm.edu.co/entities/publication/f63077de-e4b6-4938-9274-e488fafec12a
- ISO/IEC 27001 — Sistema de gestión de seguridad de la información.
- NIST SP 800-53 — Controles de seguridad y privacidad.
- Cloud Security Alliance (CSA) — Marco de seguridad en la nube.
- ISO/IEC 27005:2022 — Gestión del riesgo de seguridad de la información.

---

*Documento creado el 05/03/2026*

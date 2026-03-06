# Propuesta de Investigación — Daniel Aguirre
**Seminario 1 — Especialización en Ciberseguridad — ITM | Marzo 2026**

---

## 1. Referencias Bibliográficas

- Moncada García, A. F. (2025). *Modelo de ciberseguridad aplicado a la infraestructura como servicio IaaS usada en la nube híbrida para pymes, con base en gestión de riesgos.* Trabajo de grado — Maestría. ITM. https://hdl.handle.net/20.500.12622/8029
- ISO/IEC. (2022). *ISO/IEC 27001:2022 — Information security management systems.* ISO.
- NIST. (2020). *SP 800-53 Rev. 5: Security and Privacy Controls.* https://doi.org/10.6028/NIST.SP.800-53r5
- NIST. (2012). *SP 800-61 Rev. 2: Computer Security Incident Handling Guide.* https://doi.org/10.6028/NIST.SP.800-61r2
- Cloud Security Alliance. (2023). *Cloud Controls Matrix v4.0.* https://cloudsecurityalliance.org/research/cloud-controls-matrix
- Tounsi, W. & Rais, H. (2018). A survey on technical threat intelligence in the age of sophisticated cyber attacks. *Computers & Security*, 72, 212–233. https://doi.org/10.1016/j.cose.2017.09.001
- Schlette, D., Böhm, F., Caselli, M. & Pernul, G. (2021). Measuring and visualizing cyber threat intelligence quality. *Int. J. Inf. Secur.*, 20, 21–38. https://doi.org/10.1007/s10207-020-00490-y
- Strom, B. E. et al. (2018). *MITRE ATT&CK: Design and philosophy.* MITRE Corporation. https://attack.mitre.org
- Gartner. (2023). *Market Guide for Security Orchestration, Automation and Response Solutions.* https://www.gartner.com

---

## 2. Problema que se Necesita Resolver

La tesis de referencia diseñó un modelo de ciberseguridad para IaaS en nube híbrida basado en estándares internacionales (ISO 27001, NIST SP 800-53, CSA). Es un modelo **prescriptivo y estático**: define *qué* controles aplicar, pero **no provee los mecanismos para operarlos en tiempo real**.

El problema que esta propuesta resuelve es esa brecha operativa:

1. **Sin CTI:** los controles no se actualizan ante nuevas amenazas activas.
2. **Sin SOAR:** no hay respuesta automática ante incidentes.
3. **Sin métricas:** no es posible medir si el modelo está funcionando (MTTD, MTTR).
4. **Sin monitoreo continuo:** la vigilancia de la infraestructura IaaS no está especificada.

**Pregunta problema:**  
*¿Cómo fortalecer la tesis de referencia integrando CTI, SOAR y monitoreo continuo, para pasar de un modelo de gestión de riesgos a un modelo de defensa activa y medible?*

---

## 3. Diferencias entre la Tesis de Referencia y la Propuesta Nueva

| # | Dimensión | Tesis de referencia (2025) | Propuesta — Daniel Aguirre |
|---|---|---|---|
| 1 | **Detección de amenazas** | No incluye mecanismos de detección activa | CTI en tiempo real: controles se actualizan ante amenazas emergentes |
| 2 | **Monitoreo continuo** | Mencionado pero sin implementación concreta | SIEM integrado y especificado sobre la arquitectura IaaS híbrida |
| 3 | **Métricas de desempeño** | No define KPIs operativos | MTTD, MTTR y cobertura de controles, medibles antes/después |
| 4 | **Alcance de validación** | 1 caso de estudio | Múltiples sectores para demostrar adaptabilidad |
| 5 | **Aporte central** | *Qué* proteger y con qué controles | *Cómo* defenderse activamente y *medir* que el modelo opera |

### En una frase

> Moncada García construyó el **mapa de riesgos**.  
> Esta propuesta entrega el **sistema de defensa activa** que lo operacionaliza.

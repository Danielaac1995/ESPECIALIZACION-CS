## Evaluación DREAD de Amenazas

**DREAD** es una metodología de calificación cuantitativa de riesgo. Cada amenaza se puntúa del 1 al 10 en 5 dimensiones:
- **D** — Daño Potencial *(Damage Potential)*
- **R** — Reproducibilidad *(Reproducibility)*
- **E** — Explotabilidad *(Exploitability)*
- **A** — Usuarios Afectados *(Affected Users)*
- **D** — Descubribilidad *(Discoverability)*

**Puntuación de riesgo** = Promedio de las 5 dimensiones

| Tipo de Amenaza | Escenario | Daño Potencial | Reproducibilidad | Explotabilidad | Usuarios Afectados | Descubribilidad | Puntaje de Riesgo |
|----------------|-----------|:--------------:|:----------------:|:--------------:|:------------------:|:---------------:|:-----------------:|
| Suplantación (Spoofing) | Un atacante podría hacerse pasar por un autor legítimo de cursos de Kaggle Learn explotando vulnerabilidades en el paquete learntools, lo que podría derivar en la implementación de código malicioso o acceso no autorizado a datos de usuarios. | 9 | 8 | 7 | 9 | 6 | **7.80** 🔴 |
| Suplantación (Spoofing) | Un atacante podría manipular el sistema de plantillas de los notebooks para inyectar contenido malicioso, haciéndolo aparecer como si proviniese de un autor legítimo del curso. | 8 | 7 | 6 | 8 | 7 | **7.20** 🟠 |
| Manipulación (Tampering) | Un atacante podría modificar el código de verificación de ejercicios en el paquete learntools para alterar los materiales del curso o la retroalimentación entregada a los usuarios, introduciendo vulnerabilidades o información engañosa. | 8 | 6 | 5 | 7 | 6 | **6.40** 🟠 |
| Manipulación (Tampering) | Un atacante podría explotar vulnerabilidades en la infraestructura central del paquete learntools para alterar entregas de usuarios o resultados de ejercicios, permitiendo trampa o manipulación de datos. | 9 | 8 | 7 | 9 | 6 | **7.80** 🔴 |
| Repudio (Repudiation) | Un usuario podría negar haber completado un ejercicio o alcanzado cierta puntuación, alegando un error del sistema o que su cuenta fue comprometida. | 4 | 5 | 3 | 4 | 5 | **4.20** 🟡 |
| Divulgación de Información (Information Disclosure) | Un atacante podría explotar vulnerabilidades en el paquete learntools o en los notebooks para acceder a información sensible de usuarios, como su progreso, entregas o datos personales. | 9 | 8 | 7 | 9 | 6 | **7.80** 🔴 |
| Divulgación de Información (Information Disclosure) | Un usuario podría divulgar inadvertidamente información sensible, como credenciales de cuenta o datos personales, a través de los notebooks o entregas de ejercicios. | 7 | 6 | 5 | 6 | 7 | **6.20** 🟠 |
| Denegación de Servicio (Denial of Service) | Un atacante podría saturar el sistema con un gran número de entregas de ejercicios o solicitudes de notebooks, causando problemas de rendimiento o tiempo de inactividad. | 8 | 7 | 6 | 9 | 5 | **7.00** 🟠 |
| Denegación de Servicio (Denial of Service) | Un usuario podría accidental o intencionalmente provocar una condición de denegación de servicio enviando ejercicios o notebooks malformados, perturbando el sistema. | 6 | 5 | 4 | 6 | 6 | **5.40** 🟠 |
| Elevación de Privilegios (Elevation of Privilege) | Un atacante podría explotar vulnerabilidades en el paquete learntools o en los notebooks para obtener privilegios elevados, accediendo o modificando datos sensibles, o realizando acciones no autorizadas. | 9 | 8 | 7 | 9 | 6 | **7.80** 🔴 |
| Elevación de Privilegios (Elevation of Privilege) | Un usuario podría explotar vulnerabilidades del sistema para acceder a las cuentas o datos de otros usuarios, lo que podría llevar a acciones no autorizadas o brechas de datos. | 9 | 8 | 7 | 9 | 6 | **7.80** 🔴 |

---

### Resumen de Priorización

| Nivel | Rango | Amenazas |
|-------|-------|----------|
| 🔴 Crítico | ≥ 7.5 | Suplantación #1, Manipulación #2, Divulgación #1, Elevación #1, Elevación #2 |
| 🟠 Alto | 5.0 – 7.4 | Suplantación #2, Manipulación #1, Divulgación #2, DoS #1, DoS #2 |
| 🟡 Medio | 2.5 – 4.9 | Repudio |


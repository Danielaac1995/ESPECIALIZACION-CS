# 📓 Notas de Clase — Introducción a la Ciberseguridad

**Especialización:** Ciberseguridad  
**Docente:**  
**Lugar / Modalidad:**

---

## 📌 Sesión 1 — Ransomware: Prevención, Contención y Grupos Activos

**Fecha:** 26/02/2026

### 🛡️ Prevención

#### Autenticación Robusta — MFA y FIDO2/Passkeys

**MFA (Multi-Factor Authentication):** exige dos o más pruebas de identidad. Los factores son: algo que **sabes** (contraseña), algo que **tienes** (celular/llave física), algo que **eres** (huella/cara).

**FIDO2 / Passkeys:** estándar más seguro de MFA. Usa criptografía de clave pública vinculada al dispositivo físico. No se puede robar por phishing porque la clave nunca viaja por internet.

> El 23% de los ataques de 2025 entran por credenciales comprometidas. Con MFA, esas credenciales robadas no sirven solas.

#### Principio de Mínimo Privilegio

Cada usuario, sistema o proceso tiene acceso **solo a lo estrictamente necesario**. Limita el movimiento lateral del ransomware en la fase de Reconocimiento Lateral — el malware choca con barreras de permisos al intentar propagarse.

#### Gestión de Parches — ¿Por qué < 48h?

- El 32% de los ataques de 2025 entran por **vulnerabilidades conocidas** con parche disponible.
- Una vez publicada en el catálogo **CISA KEV**, una vulnerabilidad empieza a ser explotada masivamente en promedio en **24–72 horas**.
- **CVE:** identificador único de una vulnerabilidad (ej. CVE-2023-20269).
- **CVSS 9.0–10.0** = parchear en < 48h / **7.0–8.9** = parchear en < 7 días.

> *"El parche que no aplicas hoy es el vector del ataque de mañana."*

#### Regla de Backup 3-2-1-1-0

| Número | Significa | Por qué |
|--------|-----------|---------|
| 3 | Tres copias del dato | Si una falla, hay dos más |
| 2 | Dos tipos de medios distintos | Evita fallo de hardware único |
| 1 | Una copia offsite (fuera del sitio) | Si cifran todo en la oficina, la remota queda intacta |
| 1 | Una copia air-gapped (desconectada) | El ransomware no puede llegar ahí |
| 0 | Cero errores verificados | Un backup no probado no existe |

> El 97% de organizaciones con backups sanos se recuperó sin pagar rescate (Sophos 2025).

#### Segmentación de Red y Zero Trust

- **Segmentación:** dividir la red en zonas aisladas. El ransomware no puede saltar libremente entre zonas.
- **Zero Trust:** "nunca confiar, siempre verificar". Tres principios:
  1. Verificar siempre, aunque el usuario ya esté dentro de la red.
  2. Mínimo privilegio.
  3. Asumir compromiso — operar como si ya hubiera un intruso.

---

### 🚨 Contención

#### Aislamiento Inmediato

Desconectar el sistema comprometido de la red. **No apagar el equipo** — se pierde la memoria RAM con datos forenses clave (claves de cifrado activas, procesos del malware).

#### EDR y XDR

| | EDR | XDR |
|---|-----|-----|
| Qué es | Endpoint Detection & Response | Extended Detection & Response |
| Alcance | Un endpoint | Toda la infraestructura (red, nube, correo) |
| Detecta | Comportamientos anómalos en el dispositivo | Ataques distribuidos correlacionando señales |

El EDR puede **aislar automáticamente** un equipo sin intervención humana al detectar cifrado masivo, eliminación de shadow copies o uso de LOLBins.

#### CSIRT y COLCERT

- **CSIRT:** equipo interno de respuesta a incidentes. Los roles deben estar definidos **antes** del incidente.
- **COLCERT:** Grupo de Respuesta a Emergencias Cibernéticas de Colombia (MinTIC). Notificación obligatoria cuando afecta infraestructura crítica o datos sensibles.

#### Preservación de Evidencia

Antes de limpiar: capturar imagen forense del disco + volcado de RAM. Se busca: vector de entrada, movimiento lateral, datos exfiltrados, variante del ransomware.

#### Restauración desde Backup Limpio

El backup debe ser **anterior a la intrusión**, no solo al cifrado. El atacante estuvo en promedio **9 días** en la red antes de cifrar. Un backup de 3 días antes puede ya contener malware dormido.

---

### 🦠 Grupos más activos (2025)

| Grupo | Característica clave | Vector principal |
|-------|---------------------|-----------------|
| **Qilin** | RaaS + doble extorsión. 1.001 víctimas en 1 año | Afiliados de RansomHub |
| **Akira** | Usa AnyDesk + LOLBins. $244M extorsionados | CVE-2023-20269 (VPN Cisco) |
| **Cl0p** | Especialista en 0-days. A veces no cifra, solo exfiltra | 0-days en MOVEit, Cleo |
| **DragonForce** | Heredó infraestructura de RansomHub | Ingeniería social (Scattered Spider) |
| **Play** | Silencioso. ~900 víctimas (CISA, mayo 2025) | RDP con credenciales débiles |
| **LockBit 5.0** | Relanzó tras Operación Cronos (2024). Más víctimas históricas | Múltiples vectores |

> Conclusión clave: el ecosistema criminal es resiliente — se fragmenta pero no colapsa.

---

## 📌 Tarea — Diferencias entre Ciberseguridad, Seguridad Informática y Seguridad de la Información

**Estado:** ⬜ Pendiente de completar

| Criterio | Seguridad de la Información | Seguridad Informática | Ciberseguridad |
|---|---|---|---|
| **Objeto de protección** | | | |
| **Medio/soporte** | | | |
| **Amenazas que aborda** | | | |
| **Ámbito** | | | |
| **Estándar principal** | ISO/IEC 27001 | | NIST CSF |
| **¿Requiere internet?** | No | No | Sí |

---

*Notas actualizadas el 06/03/2026*

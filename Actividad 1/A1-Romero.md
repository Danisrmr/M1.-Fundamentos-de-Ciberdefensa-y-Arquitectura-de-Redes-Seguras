# A.1 — Entregable

**Nombre:** Romero Díaz Alma Daniela
**Fecha:** 25/09/2026  
**Archivo de capa que acompaña a este documento:** `a1-romerodiaz-scattered_spider.json`

---

## 1 · Caso elegido

**Grupo/campaña:** Scattered Spider (G1015)  
**Reporte y URL:** CISA AA23-320A — *Scattered Spider* (actualizado el 29 de julio de 2025)  
https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a

## 2 · Mapeo

| # | Táctica | Técnica (ID) | Confianza | Evidencia — frase literal del reporte |
|---|---|---|---|---|
| 1 | Reconnaissance | Search Victim-Owned Websites (T1594) | 3 — Explícita | “Scattered Spider searches business-to-business websites to gather information and ultimately determine the individual’s role in a target organization.” |
| 2 | Resource Development | Acquire Infrastructure: Domains (T1583.001) | 3 — Explícita | “Scattered Spider intrusions historically began with broad phishing and smishing attempts against a target using organization-specific crafted domains.” |
| 3 | Reconnaissance | Search Closed Sources: Purchase Technical Data (T1597.002) | 3 — Explícita | “the threat actors purchase employee or contractor credentials on illicit marketplaces such as Russia Market.” |
| 4 | Reconnaissance | Phishing for Information: Spearphishing Voice (T1598.004) | 3 — Explícita | “the threat actors then use layered social engineering techniques which frequently occur over several calls.” |
| 5 | Initial Access | Phishing: Spearphishing Voice (T1566.004) | 3 — Explícita | “the threat actors conduct spearphising calls to convince IT help desk personnel to reset passwords and/or transfer MFA tokens.” |
| 6 | Credential Access | Multi-Factor Authentication Request Generation (T1621) | 3 — Explícita | “Sent repeated MFA notification prompts leading to employees pressing the ‘Accept’ button (also known as MFA fatigue).” |
| 7 | Persistence | Modify Authentication Process: Multi-Factor Authentication (T1556.006) | 3 — Explícita | “Scattered Spider threat actors then register their own MFA tokens.” |
| 8 | Command and Control | Remote Access Tools: Remote Desktop Software (T1219.002) | 3 — Explícita | “Posed as company IT and/or helpdesk staff to direct employees to run commercial remote access tools enabling initial access.” |
| 9 | Initial Access | Valid Accounts: Domain Accounts (T1078.002) | 3 — Explícita | “the threat actors conduct spearphising calls to convince IT help desk personnel to reset passwords and/or transfer MFA tokens.” |
| 10 | Collection | Data from Information Repositories: SharePoint (T1213.002) | 3 — Explícita | “Scattered Spider threat actors often perform discovery, specifically searching for SharePoint sites.” |
| 11 | Exfiltration | Exfiltration Over Web Service: Exfiltration to Cloud Storage (T1567.002) | 3 — Explícita | “Recently, this includes exfiltration to multiple sites including MEGA[.]NZ and U.S.-based data centers such as Amazon S3.” |
| 12 | Impact | Data Encrypted for Impact (T1486) | 3 — Explícita | “encrypt data on the system for ransom.” |

### Criterio de confianza

- **3 — Explícita:** el comportamiento está descrito de forma directa en el reporte.
- **2 — Implícita:** el comportamiento requiere una inferencia razonable a partir de la evidencia.
- **1 — Inferida:** la relación es posible, pero el reporte no la establece con suficiente claridad.

En esta selección, **las 12 técnicas tienen confianza 3**, porque todas cuentan con comportamiento explícito en el reporte. En once casos, el advisory además las identifica directamente con el mismo ID. La única particularidad es **T1219.002**: el advisory, basado en ATT&CK v17, utiliza el ID padre **T1219 — Remote Access Software**, mientras que Enterprise ATT&CK v19 divide esa conducta en sub-técnicas. La evidencia del reporte especifica el uso de software comercial de acceso remoto/RMM y, por ello, permite seleccionar de forma explícita **T1219.002 — Remote Desktop Software** en v19. 

## 3 · Confrontación con el mapeo oficial

Para la comparación tomé como referencia las tablas ATT&CK incluidas en el reporte CISA. Como la actividad se realiza en **Enterprise ATT&CK**, no conté como omisiones las técnicas de dominio Mobile **T1660** y **T1451**.

| | Cuántas | Cuáles |
|---|---:|---|
| **Aciertos** (tú y el reporte) | 12 | T1594, T1583.001, T1597.002, T1598.004, T1566.004, T1621, T1556.006, T1078.002, T1213.002, T1567.002, T1486 y la correspondencia T1219 (v17) → T1219.002 (v19). |
| **Omisiones** (el reporte sí, tú no) | 29 | T1589, T1598, T1593.001, T1585.001, T1566, T1199, T1648, T1204, T1136, T1078, T1484.002, T1578.002, T1656, T1606, T1552.001, T1552.004, T1217, T1538, T1083, T1018, T1539, T1021.007, T1213.003, T1074, T1114, T1530, T1090, T1567 y T1657. |
| **Extras** (tú sí, el reporte no) | 0 | No hay extras de comportamiento. T1219.002 es el refinamiento en v19 del comportamiento que el reporte v17 representa con T1219. |

El grupo de omisiones fue el más grande porque mi capa resume los comportamientos que consideré más representativos y con evidencia clara; no intenta reproducir todas las técnicas mencionadas en el advisory. Los doce aciertos muestran que las técnicas seleccionadas están respaldadas directamente por el reporte. La diferencia entre T1219 y T1219.002 se debe a la versión de ATT&CK utilizada y no a una inferencia adicional sobre el comportamiento.

La capa oficial del grupo Scattered Spider es más extensa porque representa comportamientos acumulados de múltiples incidentes y periodos. Por eso no es esperable que una capa construida a partir de un solo reporte tenga el mismo tamaño que el perfil completo del grupo.

## 4 · La técnica más difícil de mitigar

La técnica que considero más difícil de mitigar es **Phishing: Spearphishing Voice (T1566.004)**, porque explota principalmente la confianza y los procedimientos humanos del help desk. Un firewall por sí solo no evita que una persona sea convencida de restablecer una contraseña o transferir un factor de autenticación. Para reducir el riesgo se necesitan procedimientos estrictos de verificación de identidad, devolución de llamada por canales registrados, aprobación adicional para cambios sensibles y capacitación continua. También ayuda utilizar MFA resistente al phishing para disminuir el impacto si la ingeniería social tiene éxito.

## 5 · Pitch de 90 segundos a la dirección

Este caso muestra que el acceso inicial puede comenzar con una llamada y no necesariamente con una vulnerabilidad técnica. Los atacantes se hicieron pasar por personal legítimo, manipularon procesos de soporte y lograron obtener credenciales o control sobre la autenticación. Después utilizaron herramientas legítimas de acceso remoto, buscaron información sensible y llegaron a exfiltrar y cifrar datos. Esto significa que proteger la organización requiere combinar controles técnicos con procesos sólidos de identidad y soporte. Propongo reforzar la verificación de identidad del help desk, limitar el software de acceso remoto y controlar mejor el acceso a sistemas y datos sensibles. La decisión que solicito es priorizar estas medidas como controles de negocio y no únicamente como configuraciones de TI.

## 6 · Puente al entregable final del módulo

| Técnica (ID) | Mitigación ATT&CK | Qué cambiaría yo en la topología |
|---|---|---|
| Remote Access Tools: Remote Desktop Software (T1219.002) | M1037 — Filter Network Traffic | Forzaría el tráfico de salida de los equipos por firewall/proxy y permitiría únicamente los servicios de administración remota autorizados. El tráfico hacia servicios RMM no aprobados quedaría bloqueado o restringido. |
| Data from Information Repositories: SharePoint (T1213.002) | M1035 — Limit Access to Resource Over Network | Separaría los repositorios sensibles de los segmentos de usuario y permitiría su acceso solamente desde identidades, dispositivos y redes autorizadas, aplicando mínimo privilegio. |
| Valid Accounts: Domain Accounts (T1078.002) | M1030 — Network Segmentation | Separaría administración, identidad, usuarios y servidores críticos en zonas distintas con reglas explícitas entre segmentos, de modo que una cuenta comprometida no otorgue acceso amplio a toda la red. |

Estas decisiones no eliminan por sí solas las técnicas, pero reducen las rutas disponibles para el adversario y el alcance que podría tener una cuenta o un equipo comprometido.

## 7 · Pregunta guía

ATT&CK Navigator comunica de forma visual en qué etapas se concentra el comportamiento del adversario y permite distinguir rápidamente qué técnicas tienen evidencia más fuerte mediante los scores y colores. Un documento de texto puede explicar cada acción con más detalle, pero la capa facilita ver el patrón completo, las concentraciones y los huecos del análisis en una sola vista.

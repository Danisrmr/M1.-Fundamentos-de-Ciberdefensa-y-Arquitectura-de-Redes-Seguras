# A.1 — Entregable

**Nombre:** Romero Díaz Alma Daniela
**Fecha:** 25/09/2026  
**Archivo de capa que acompaña a este documento:** `a1-romerodiaz-scattered_spider.json`

---

## 1 · Caso elegido

**Grupo/campaña:** Scattered Spider (G1015)  
**Reporte y URL:** CISA AA23-320A — *Scattered Spider*
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
| 9 | Privilege Escalation | Valid Accounts: Domain Accounts (T1078.002) | 3 — Explícita | “the threat actors conduct spearphising calls to convince IT help desk personnel to reset passwords and/or transfer MFA tokens.” |
| 10 | Collection | Data from Information Repositories: SharePoint (T1213.002) | 3 — Explícita | “Scattered Spider threat actors often perform discovery, specifically searching for SharePoint sites.” |
| 11 | Exfiltration | Exfiltration Over Web Service: Exfiltration to Cloud Storage (T1567.002) | 3 — Explícita | “Recently, this includes exfiltration to multiple sites including MEGA[.]NZ and U.S.-based data centers such as Amazon S3.” |
| 12 | Impact | Data Encrypted for Impact (T1486) | 3 — Explícita | “encrypt data on the system for ransom.” |

Para asignar la confianza utilicé la escala indicada en la actividad. En este caso marqué las 12 técnicas con nivel 3 porque el comportamiento se menciona directamente en el reporte y no tuve que suponer que ocurrió.

La única técnica que necesitó una pequeña adaptación fue T1219.002. El reporte de CISA utiliza una versión anterior de ATT&CK y la presenta como T1219, mientras que en la versión utilizada en Navigator aparece de forma más específica como T1219.002. El comportamiento sigue siendo el mismo: el uso de herramientas de acceso remoto.


## 3 · Confrontación con el mapeo oficial
| | Cuántas | Cuáles |
|---|---:|---|
| **Aciertos** (tú y el reporte) | 12 | T1594, T1583.001, T1597.002, T1598.004, T1566.004, T1621, T1556.006, T1078.002, T1213.002, T1567.002, T1486 y T1219/T1219.002. |
| **Omisiones** (el reporte sí, tú no) | 29 | T1589, T1598, T1593.001, T1585.001, T1566, T1199, T1648, T1204, T1136, T1078, T1484.002, T1578.002, T1656, T1606, T1552.001, T1552.004, T1217, T1538, T1083, T1018, T1539, T1021.007, T1213.003, T1074, T1114, T1530, T1090, T1567 y T1657. |
| **Extras** (tú sí, el reporte no) | 0 | No identifiqué técnicas adicionales que no estuvieran respaldadas por el reporte. |

El grupo más grande fue el de omisiones. Esto pasó porque el reporte contiene muchas más técnicas de las que seleccioné para mi capa. Yo me enfoqué en las que me parecieron más claras y representativas para explicar cómo trabaja Scattered Spider. Aunque no incluí todo lo que aparece en el reporte, las técnicas que seleccioné sí tienen evidencia directa.

También observé que el perfil completo de Scattered Spider en MITRE contiene muchas más técnicas. Esto tiene sentido porque ese perfil reúne información de diferentes incidentes y momentos, mientras que mi capa está basada principalmente en el reporte que analicé.


## 4 · La técnica más difícil de mitigar

Considero que una de las técnicas más difíciles de mitigar es **Phishing: Spearphishing Voice (T1566.004)**. El problema es que no depende únicamente de una falla en un sistema, sino de convencer a una persona para que realice una acción. Por ejemplo, los atacantes pueden hacerse pasar por un empleado y llamar al help desk para pedir un cambio de contraseña o de MFA.

Para disminuir este riesgo sería necesario tener procesos más estrictos para verificar la identidad de los usuarios antes de hacer cambios importantes. También ayudaría capacitar al personal y utilizar métodos de autenticación más resistentes al phishing.

## 5 · Pitch de 90 segundos a la dirección

El caso de Scattered Spider demuestra que un ataque no siempre empieza aprovechando una vulnerabilidad técnica. En este caso, una parte importante del ataque se basa en engañar a empleados y personal de soporte para conseguir acceso a las cuentas.

Después de obtener acceso, los atacantes pueden utilizar herramientas legítimas de administración remota, buscar información dentro de la organización y extraer datos. En algunos casos también pueden cifrar la información para pedir un rescate.

Por esta razón, considero importante reforzar los procesos de verificación de identidad, controlar mejor las herramientas de acceso remoto y limitar el acceso a información sensible. La seguridad no debería depender solamente del firewall, sino también de los procesos que siguen los empleados y el área de soporte.

## 6 · Puente al entregable final del módulo

| Técnica (ID) | Mitigación ATT&CK | Qué cambiaría yo en la topología |
|---|---|---|
| Remote Access Tools: Remote Desktop Software (T1219.002) | M1037 — Filter Network Traffic | Limitaría desde el firewall o proxy las herramientas de acceso remoto que pueden utilizarse. Solo permitiría aquellas que hayan sido autorizadas por la organización. |
| Data from Information Repositories: SharePoint (T1213.002) | M1035 — Limit Access to Resource Over Network | Separaría los recursos que contienen información sensible y permitiría que solamente usuarios o equipos autorizados puedan acceder a ellos. |
| Valid Accounts: Domain Accounts (T1078.002) | M1030 — Network Segmentation | Dividiría la red en diferentes segmentos para evitar que una cuenta comprometida tenga acceso directo a todos los sistemas de la organización. |

Estas medidas no impedirían por completo que las técnicas fueran utilizadas, pero sí podrían limitar el acceso del atacante y reducir el daño que podría causar dentro de la red.

## 7 · Pregunta guía

ATT&CK Navigator permite observar de manera visual en qué partes del ataque se concentran las técnicas utilizadas. Esto facilita identificar rápidamente los comportamientos más importantes y comparar diferentes etapas del ataque.

En un documento de texto también se puede explicar esta información, pero con Navigator es más fácil tener una vista general y entender la relación entre las técnicas y las tácticas.

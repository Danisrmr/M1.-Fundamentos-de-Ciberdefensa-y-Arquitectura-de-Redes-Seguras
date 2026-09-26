# A.1 - ATT&CK Navigator

## Del relato al mapa: Scattered Spider

**Alumno(a):** Daniela Romero  
**Caso elegido:** Scattered Spider (G1015)  
**Reporte:** CISA AA23-320A - Scattered Spider  
**Enlace:** https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a  
**Versión de trabajo:** MITRE ATT&CK Enterprise v19 / Navigator 5.3.2

## 1. Caso elegido y enlace al reporte

Elegí el caso **Scattered Spider**, un actor que combina ingeniería social, abuso de procesos de mesa de ayuda, manipulación de MFA, herramientas legítimas de acceso remoto, movimiento en servicios cloud, exfiltración y extorsión. El análisis se limita a las conductas respaldadas por el reporte de CISA y las normaliza a ATT&CK Enterprise v19.

## 2. Tabla de mapeo con la evidencia citada

**Escala de confianza:** 3 = explícita; 2 = implícita con alta confianza; 1 = inferida. No se incluyeron técnicas con score 1 porque se prefirió no afirmar conductas que el reporte no sustenta.

| # | Técnica ATT&CK v19 | Táctica | Confianza | Evidencia literal del reporte | Justificación |
|---:|---|---|:---:|---|---|
| 1 | T1583.001 - Acquire Infrastructure: Domains | Resource Development | 3 | “using organization-specific crafted domains” | El uso de dominios creados/adquiridos para apoyar el engaño encaja directamente con Domains. |
| 2 | T1598.004 - Phishing for Information: Spearphishing Voice | Reconnaissance | 3 | “phone calls to employees and help desks to gather password reset specific information” | El objetivo de la llamada es recolectar información accionable mediante vishing. |
| 3 | T1566.004 - Phishing: Spearphishing Voice | Initial Access | 3 | “conduct spearphising calls to convince IT help desk personnel to reset passwords and/or transfer MFA tokens” | La voz se utiliza para obtener acceso inicial, no solo para recolectar información. |
| 4 | T1684.001 - Social Engineering: Impersonation | Stealth | 3 | “Posed as company IT and/or helpdesk staff” | En ATT&CK v19 T1656 fue revocada en favor de T1684.001; la conducta de suplantación es explícita. |
| 5 | T1621 - Multi-Factor Authentication Request Generation | Credential Access | 3 | “Sent repeated MFA notification prompts leading to employees pressing the ‘Accept’ button” | Es el patrón de MFA fatigue descrito directamente por T1621. |
| 6 | T1556.006 - Modify Authentication Process: Multi-Factor Authentication | Defense Impairment / Persistence / Credential Access | 3 | “register their own MFA tokens” | Modificar el método MFA permite persistencia y reduce el valor del control original. |
| 7 | T1219.002 - Remote Access Tools: Remote Desktop Software | Command and Control | 2 | “deploy remote monitoring and management (RMM) tools ... to establish persistence” | La conducta es explícita; el score 2 refleja que el reporte v17 usaba el padre T1219 y aquí se refina a la sub-técnica v19 T1219.002. |
| 8 | T1213.002 - Data from Information Repositories: SharePoint | Collection | 3 | “searching for SharePoint sites” | El repositorio y la acción aparecen expresamente en el reporte. |
| 9 | T1114 - Email Collection | Collection | 3 | “search ... Microsoft Exchange Online for emails ... regarding the threat actors’ intrusion and any security response” | La búsqueda de mensajes de correo es una forma directa de Email Collection. |
| 10 | T1021.007 - Remote Services: Cloud Services | Lateral Movement | 3 | “then move to both preexisting ... Amazon Elastic Compute Cloud (EC2) instances” | El movimiento hacia instancias EC2 preexistentes está explícitamente descrito. |
| 11 | T1567.002 - Exfiltration Over Web Service: Exfiltration to Cloud Storage | Exfiltration | 3 | “exfiltration to multiple sites including MEGA[.]NZ ... and Amazon S3” | MEGA y S3 son destinos de almacenamiento cloud usados para exfiltración. |
| 12 | T1486 - Data Encrypted for Impact | Impact | 3 | “encrypt data on the system for ransom” | La acción descrita coincide directamente con cifrado de datos para impacto. |

### Nota de normalización v17 -> v19

El reporte CISA fue redactado con ATT&CK v17. Para esta entrega se usa v19, como pide la actividad. Por eso, la conducta de **Impersonation** que el reporte vincula con T1656 se representa como **T1684.001 - Social Engineering: Impersonation**, porque T1656 fue revocada en ATT&CK v19 en favor de T1684.001. Además, para el comportamiento de RMM se eligió **T1219.002 - Remote Desktop Software**, una sub-técnica v19 más específica que el T1219 general utilizado por el reporte.

## 3. Confrontación: aciertos / omisiones / extras + qué te dice

**Aciertos.** La comparación con la capa oficial de Scattered Spider confirma varias coincidencias centrales: T1621 (MFA fatigue), T1556.006 (registro de MFA propio), T1566.004 y T1598.004 (vishing), T1684.001 (impersonación), T1219.002 (RMM), T1213.002 (SharePoint), T1114 (correo), T1021.007 (cloud), T1567.002 (exfiltración a nube) y T1486 (cifrado para impacto).

**Omisiones.** El perfil oficial del grupo es mucho más amplio porque agrega técnicas observadas en distintas campañas y periodos. Mi capa no intenta reproducir todo ese historial. Dejé fuera, entre otras, técnicas como T1594 (búsqueda en sitios de la víctima), T1199 (Trusted Relationship), T1083 (File and Directory Discovery), T1090 (Proxy) y T1136 (Create Account), aunque algunas también aparecen en el reporte.

**Extras.** No agregué comportamientos sin respaldo textual. Las diferencias principales no son “extras” de conducta, sino normalizaciones de versión: T1656 -> T1684.001 y T1219 -> T1219.002.

**Qué me dice la confrontación.** Una capa de incidente y una capa de grupo tienen escalas distintas. La primera explica un relato concreto; la segunda acumula conocimientos históricos. Por eso una omisión frente al perfil completo no implica necesariamente un error si la técnica no es necesaria para explicar el caso que se está analizando.

## 4. La técnica que me parece más difícil de mitigar y por qué

La técnica más difícil de mitigar en este caso es **T1684.001 - Social Engineering: Impersonation**. El atacante explota confianza, urgencia y procedimientos normales de soporte, por lo que un firewall no puede distinguir por sí solo si la persona que llama realmente pertenece a TI. Además, la suplantación se combina con información previa de la víctima y con solicitudes aparentemente legítimas, como restablecer contraseñas o mover un factor MFA. Su mitigación requiere controles técnicos y, sobre todo, procesos: verificación independiente de identidad, procedimientos estrictos del help desk, MFA resistente al phishing, capacitación y alertas ante cambios sensibles.

## 5. Pitch de 90 segundos a un director no técnico

Scattered Spider demuestra que una intrusión grave puede comenzar sin explotar una vulnerabilidad técnica: el atacante convence a una persona de que es parte de TI o incluso se hace pasar por un empleado. A partir de ahí obtiene restablecimientos de contraseña, provoca fatiga de MFA y registra factores bajo su control. Después usa herramientas legítimas de acceso remoto, se desplaza hacia servicios cloud, busca información sensible y puede extraer datos a servicios como MEGA o Amazon S3. En algunos incidentes también cifra sistemas para exigir un rescate. La prioridad defensiva no debe ser solo comprar más tecnología; hay que reforzar los procesos de identidad y mesa de ayuda, limitar herramientas de acceso remoto, segmentar los recursos críticos y controlar el tráfico de salida. ATT&CK Navigator permite ver ese recorrido completo y decidir en qué etapas existen controles y en cuáles quedan huecos.

## 6. Las 3 técnicas que una decisión de arquitectura de red podría afectar

1. **T1219.002 - Remote Desktop Software.** M1037 (Filter Network Traffic) puede limitar salidas hacia servicios de RMM no autorizados y M1031 (Network Intrusion Prevention) puede detectar o bloquear tráfico asociado con servicios remotos conocidos.
2. **T1021.007 - Cloud Services.** Los principios de M1030 (Network Segmentation) y M1035 (Limit Access to Resource Over Network) pueden reducir las rutas y recursos a los que una identidad comprometida puede llegar, separando planos de administración, redes y recursos críticos.
3. **T1567.002 - Exfiltration to Cloud Storage.** Una política de egreso basada en M1037 puede restringir destinos o servicios de almacenamiento no aprobados y obligar el tráfico web a pasar por controles de inspección. ATT&CK asocia además esta sub-técnica con controles específicos de contenido web; aquí se resalta M1037 porque es una de las mitigaciones pedidas por la actividad.

**Límite importante:** estas decisiones de red ayudan sobre todo después de que el atacante consiguió acceso. No evitan por sí solas una llamada de vishing o que una mesa de ayuda transfiera MFA al atacante.

## 7. Pregunta guía

**¿Qué comunica esta herramienta que un documento de texto no comunica con la misma claridad?**

ATT&CK Navigator convierte el relato en una vista visual por tácticas, de modo que se aprecia rápidamente dónde se concentra la actividad del adversario y qué fases cubre. Además, el color y el score permiten distinguir la confianza del análisis y comparar una capa de incidente con una capa oficial sin leer nuevamente todo el reporte.

## 8. Configuración de la capa de Navigator

- Nombre: `A1-Romero-ScatteredSpider`
- Dominio: Enterprise ATT&CK
- ATT&CK: v19
- Navigator: 5.3.2
- Técnicas seleccionadas: 12
- Tácticas cubiertas: más de 4 (incluye Reconnaissance, Resource Development, Initial Access, Stealth, Credential Access, Persistence, Defense Impairment, Command and Control, Collection, Lateral Movement, Exfiltration e Impact).
- Gradiente: mínimo 1, máximo 3.
- Leyenda: `3 - explícita`, `2 - implícita`, `1 - inferida`.
- Cada técnica contiene comentario con evidencia y justificación.

## 9. Qué NO hace ATT&CK Navigator

Navigator **no detecta, bloquea ni prueba** que un ataque haya ocurrido. Tampoco convierte el score en severidad o probabilidad. Es una herramienta de visualización y análisis que ayuda a representar técnicas, comparar capas y comunicar cobertura o razonamiento de forma estructurada.

## 10. Referencias

- Cybersecurity and Infrastructure Security Agency (CISA). *Scattered Spider*. AA23-320A. https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-320a
- MITRE ATT&CK. *Scattered Spider (G1015).* https://attack.mitre.org/groups/G1015/
- MITRE ATT&CK. *Official Scattered Spider Enterprise Navigator Layer.* https://attack.mitre.org/groups/G1015/G1015-enterprise-layer.json
- MITRE ATT&CK. *Social Engineering: Impersonation (T1684.001).* https://attack.mitre.org/techniques/T1684/001/
- MITRE ATT&CK. *Phishing: Spearphishing Voice (T1566.004).* https://attack.mitre.org/techniques/T1566/004/
- MITRE ATT&CK. *Phishing for Information: Spearphishing Voice (T1598.004).* https://attack.mitre.org/techniques/T1598/004/
- MITRE ATT&CK. *Multi-Factor Authentication Request Generation (T1621).* https://attack.mitre.org/techniques/T1621/
- MITRE ATT&CK. *Modify Authentication Process: Multi-Factor Authentication (T1556.006).* https://attack.mitre.org/techniques/T1556/006/
- MITRE ATT&CK. *Remote Access Tools: Remote Desktop Software (T1219.002).* https://attack.mitre.org/techniques/T1219/002/
- MITRE ATT&CK. *Exfiltration Over Web Service: Exfiltration to Cloud Storage (T1567.002).* https://attack.mitre.org/techniques/T1567/002/
- MITRE ATT&CK. *Data Encrypted for Impact (T1486).* https://attack.mitre.org/techniques/T1486/

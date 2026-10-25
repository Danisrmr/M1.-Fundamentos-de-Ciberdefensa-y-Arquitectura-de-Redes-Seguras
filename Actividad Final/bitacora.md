# Bitácora de prueba de penetración — Delphos

**Pentester:** Alma Daniela Romero Díaz
**Fecha:** 17–24 de octubre de 2026 
**Alcance:** únicamente `192.168.59.10` | **Laboratorio:** red host-only `192.168.59.0/24`  
**Autorización:** Laboratorio aislado; carta de autorización de la práctica 
**Clasificación:** Confidencial - contiene datos sensibles ficticios.


| Fecha/hora | Fase | Comando o acción exactos | Qué se buscaba | Resultado observado | Evidencia |
|---|---|---|---|---|---|
| 18-19/oct | Preparación | `vagrant up`; `vagrant ssh -c "sudo docker ps"` | Comprobar servidor y servicios | Delphos activo, Nginx, MariaDB, Redis, Tomcat y Jenkins en estado Up | E-01 |
| 18-19/oct | Preparación | `ping -c 4 192.168.59.10` | Verificar conectividad | Respuestas de Delphos, sin pérdida | E-02 |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/` | Portada | Sitio ficticio Delphos, HTML visible y metadatos de CMS | E-03 |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/robots.txt` | Rutas publicadas | `Disallow: /backups/`, `/config/`, `/portal-legacy/` | E-04 |
| 18-19/oct | 1. Reconocimiento | `whatweb http://192.168.59.10/` | Tecnologías | nginx 1.27.5 y CMS Delphos v1.2 | E-03 / E-06 |
| 18-19/oct | 1. Reconocimiento | `curl -i http://192.168.59.10/backups/` | Índice de respaldos | HTTP 200; enlaces `config.old` y `notas-migracion.txt` | E-05 |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/backups/config.old` | Configuración expuesta | Parámetros y credenciales ficticias; **no reproducidas aquí** | E-05 (índice; contenido reservado) |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/backups/notas-migracion.txt` | Notas de migración | Coexistencia de servicios y riesgos de configuración descritos | E-05 (índice; notas en el registro local) |
| 19/oct | 1. Reconocimiento | `curl -i http://192.168.59.10/config/` y `curl -i http://192.168.59.10/portal-legacy/` | Rutas sugeridas | Ambas respondieron 404; no afirmar que estuvieran publicadas | E-04 (rutas citadas; 404 documentados en registro local) |
| 18-19/oct | 2. Escaneo | `sudo nmap -sS -sV -p- -T4 192.168.59.10 -oN evidencias/09-nmap-completo.txt` | Inventario completo TCP | Puertos abiertos 22, 80, 3306, 6379, 8080, 8083; duración 4760.29 segundos | E-06 |
| 19/oct | 2. Escaneo | `sudo nmap -sC -sV -p 22,80,3306,6379,8080,8083 192.168.59.10 -oN evidencias/10-nmap-detalle.txt` | Enumerar versiones y scripts | OpenSSH 9.6p1, nginx 1.27.5, MariaDB 10.5.29, Redis 5.0.14, Tomcat 9.0.30 y Jetty/Jenkins | E-06 |
| 19/oct | 3. Enumeración | `redis-cli -h 192.168.59.10 -p 6379 PING` | Redis | `PONG` sin contraseña | E-07 |
| 19/oct | 3. Enumeración | `mariadb --disable-ssl -h 192.168.59.10 -P 3306 -u delphos_app -p` | MariaDB | Conexión de usuario de app, base `delphos` visible; sin extracción masiva | E-12 (acceso posterior; sesión de app en registro local) |
| 19/oct | 3. Enumeración | `curl -i http://192.168.59.10:8080/manager/html` | Tomcat Manager | HTTP 401 con Basic Auth; luego panel visible en navegador | E-08 |
| 19/oct | 3. Enumeración | `curl -I http://192.168.59.10:8083/` | Jenkins | HTTP 200; versión 2.190.1 en cabeceras | E-09 |
| 24/oct | 4. OpenVAS | Target `Delphos`, Task `Pentest Delphos`, perfil `Full and fast`, OpenVAS Default | Evaluar vulnerabilidades | Task `Done`; 106 resultados brutos, 6 incluidos con filtro QoD >= 70 | E-10 / openvas-delphos.pdf |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/mysql/mysql_version`; `set RHOSTS 192.168.59.10`; `set RPORT 3306`; `run` | MariaDB | Identificación de MariaDB 10.5.29 | E-06 / E-11 |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/redis/redis_server`; `set RHOSTS 192.168.59.10`; `set RPORT 6379`; `set COMMAND INFO`; `set THREADS 1`; `run` | Redis | `Found redis with INFO command`, versión 5.0.14 | E-13 |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/ssh/ssh_version`; `set RHOSTS 192.168.59.10`; `set RPORT 22`; `run` | SSH | OpenSSH 9.6p1; MAC `umac-64` ofrecidos | E-06 / E-14 |
| 24/oct | 5. Validación | `use auxiliary/scanner/mysql/mysql_login`; `run mysql://root:[CLAVE_OCULTA]@192.168.59.10:3306 createsession=true` | MariaDB | Acceso root exitoso y sesión MySQL abierta | E-11 |
| 24/oct | 5. Validación | `sessions -i 1`; `query 'SELECT CURRENT_USER(), VERSION();'`; `query 'SHOW DATABASES;'` | MariaDB | `root@%`, MariaDB 10.5.29, base delphos | E-12 |
| 24/oct | 5. Validación | `query 'SELECT COUNT(*) AS total_clientes FROM delphos.clientes;'`; `query 'SELECT COUNT(*) AS total_usuarios FROM delphos.usuarios_portal;'` | MariaDB | **20 clientes y 4 usuarios** (conteos, sin datos de filas) | E-12 |
| 24/oct | 5. Validación | `use auxiliary/scanner/ssh/ssh_login` con `KEY_PATH=/home/kali/pentest-delphos/ssh-keys/vagrant-lab`, `set STOP_ON_SUCCESS true`, `set CreateSession true`; `run` | SSH | Autenticación reportada como exitosa; fallo al establecer shell interactiva (`Unknown platform`/`fd for nil`) | E-14 (éxito directo; fallo de sesión en registro local) |
| 24/oct | 5. Validación | `ssh -i ~/pentest-delphos/ssh-keys/vagrant-lab -o IdentitiesOnly=yes -o BatchMode=yes ubuntu@192.168.59.10 'id; whoami; hostname'` | SSH | Ejecución remota como **ubuntu, uid 1001**, servidor `srv-delphos` | E-14 |

## Índice de evidencias numeradas (anexo del informe PDF)

| ID | Fase | Captura y resultado identificado |
|---|---|---|
| E-01 | Preparación | Cinco contenedores Docker activos. La consulta desde Vagrant muestra Nginx, MariaDB, Redis, Tomcat y Jenkins en ejecución. |
| E-02 | Preparación | Conectividad Kali-Delphos. Interfaz Kali en la red host-only y respuesta de la IP 192.168.59.10. |
| E-03 | Reconocimiento | Página pública de Aseguradora Delphos. Portal de prueba accesible mediante HTTP en el puerto 80. |
| E-04 | Reconocimiento | Archivo robots.txt. Las rutas publicadas incluyen /backups/, /config/ y /portal-legacy/. |
| E-05 | Reconocimiento | Índice de respaldos accesible. Se identifican /backups/, config.old y notas-migracion.txt. Se omite el contenido con secretos. |
| E-06 | Escaneo | Nmap: puertos abiertos y versiones. Seis puertos TCP abiertos y versiones asociadas a SSH, web, base de datos, caché y gestores. |
| E-07 | Enumeración | Redis respondió PONG. Se aceptó la consulta de disponibilidad sin presentar credenciales. |
| E-08 | Enumeración | Acceso al gestor de Tomcat. Se observa la consola Tomcat Web Application Manager del laboratorio. |
| E-09 | Enumeración | Panel de Jenkins visible. Panel accesible sin pantalla previa de autenticación; no prueba todos los permisos. |
| E-10 | OpenVAS/GVM | Resultados críticos del escaneo. Se muestran dos hallazgos críticos señalados por el reporte automatizado. |
| E-11 | Validación | Metasploit: autenticación administrativa en MariaDB. Una sesión MySQL fue abierta después de autenticar como root; credencial cubierta. |
| E-12 | Validación | MariaDB: conteos sin extraer registros. Consulta SELECT COUNT(*) de 20 clientes y 4 usuarios, sin visualizar información individual. |
| E-13 | Validación | Redis: comando INFO mediante Metasploit. El módulo reporta que el servicio aceptó INFO y devolvió metadatos. |
| E-14 | Validación | SSH: ejecución de comandos como ubuntu. La conexión SSH responde id, whoami y hostname; uid 1001 y srv-delphos. |


## Observaciones necesarias para reproducibilidad

- Se verificó que Delphos estuviera levantada con `vagrant up`, y que los cinco contenedores Docker estuvieran en estado `Up`.
- El escaneo OpenVAS terminó el 24/oct/2026 de 04:52:59 a 05:37:43 UTC; su PDF muestra **2 críticos, 1 alto, 2 medios, 1 bajo** con filtro `min_qod=70`. No representa todos los 106 registros.
- No se obtuvo una shell interactiva funcional desde Metasploit SSH: `sessions -i 2` mostró el error `undefined method fd for nil`. La autenticación se aceptó, pero la ejecución remota se probó **con el cliente `ssh` de Kali**.
- La salida `uid=1000(kali)` de un `id` ejecutado en la consola local de Metasploit **no es** prueba de control de Delphos; se excluyó de los hallazgos.
- Los 404 de `/config/` y `/portal-legacy/` no demuestran exposición de dichas rutas.
- No se realizaron `DELETE`, `DROP`, `UPDATE`, despliegues WAR ni creación de jobs para demostrar los hallazgos.
- **Credenciales en evidencias:** evitar publicar el archivo `config.old` y la clave privada completa presente en el PDF original. Guardar archivos sin secretos en anexos destinados al aula.


## Registro complementario por intento

### Intento 01 — Acceso a MariaDB con cifrado solicitado por el cliente

- **Fecha y hora:** 19 de octubre de 2026 (hora no documentada).
- **Fase:** 3. Enumeración.
- **Objetivo y puerto:** `192.168.59.10:3306`.
- **Comando exacto / pasos de interfaz:** `mariadb -h 192.168.59.10 -P 3306 -u delphos_app -p`.
- **Resultado literal o resumen verificable:** `ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it`.
- **Conclusión:** El intento falló por incompatibilidad TLS del cliente; **no** significa que el usuario o el servicio fueran inaccesibles.
- **Archivo de evidencia:** registro auxiliar de MariaDB del 19/oct (sin imagen numerada específica; E-12 documenta el acceso posterior).
- **Impacto para Delphos:** La configuración TLS debe corregirse de forma segura; el acceso del usuario `delphos_app` se verificó posteriormente con `--disable-ssl` dentro de la red aislada.
- **Remediación / decisión de arquitectura:** habilitar TLS correctamente, no usar `--disable-ssl` en operación real; mover la BD a VLAN de datos.

### Intento 02 — Credencial administrativa de MariaDB

- **Fecha y hora:** 24 de octubre de 2026, alrededor de 11:56 (hora mostrada en terminal).
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:3306`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/mysql/mysql_login`; `run mysql://root:[CLAVE_OCULTA]@192.168.59.10:3306 createsession=true`.
- **Resultado literal o resumen verificable:** `Success`, sesión MySQL abierta; usuario `root@%`, `delphos` visible, 20 filas de `clientes` y 4 de `usuarios_portal` contadas con `COUNT(*)`.
- **Conclusión:** Confirmado acceso administrativo **a MariaDB**; no demuestra privilegios root del SO.
- **Archivo de evidencia:** E-11 y E-12 (en el anexo del informe); registro auxiliar `16-validacion-mariadb.txt`.
- **Impacto para Delphos:** Riesgo de confidencialidad e integridad de registros si el servicio estuviera en producción.
- **Remediación / decisión de arquitectura:** rotar credencial, cuenta mínima, limitar 3306 a servidores autorizados.

### Intento 03 — Redis sin autenticación

- **Fecha y hora:** 24 de octubre de 2026 (hora no documentada).
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:6379`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/redis/redis_server`; `set RHOSTS 192.168.59.10`; `set RPORT 6379`; `set COMMAND INFO`; `run`.
- **Resultado literal o resumen verificable:** `Found redis with INFO command`, versión 5.0.14.
- **Conclusión:** Confirmada consulta de información sin contraseña. No se probaron comandos de escritura ni lectura de claves.
- **Archivo de evidencia:** E-13 (en el anexo del informe); registro auxiliar `15-metasploit.txt`.
- **Impacto para Delphos:** Exposición de información operativa de la caché; riesgo adicional si se permitiesen otros comandos.
- **Remediación / decisión de arquitectura:** ACL de Redis, acceso desde VLAN de aplicación, firewall y TLS según despliegue.

### Intento 04 — Sesión SSH en Metasploit y validación directa

- **Fecha y hora:** 24 de octubre de 2026, alrededor de 12:18 (terminal).
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:22`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/ssh/ssh_login`; `set USERNAME root`; `set KEY_PATH /home/kali/pentest-delphos/ssh-keys/vagrant-lab`; `run`; `sessions -l`; `sessions -i 2`.
- **Resultado literal o resumen verificable:** El módulo informó `Success`, pero falló la creación/manipulación de sesión con `Unknown platform` y `undefined method fd for nil`. La instrucción `id` posterior se ejecutó *localmente* en Kali, por lo que no se acepta como prueba de Delphos.
- **Conclusión:** Éxito de autenticación reportado, **sin** consola interactiva funcional en Metasploit. Se validó después con `ssh -i ... ubuntu@192.168.59.10 'id; whoami; hostname'`, obteniendo `ubuntu`, `uid=1001` y `srv-delphos`.
- **Archivo de evidencia:** E-14 (en el anexo del informe); los errores de Metasploit permanecen en `17-validacion-ssh.txt` como registro auxiliar.
- **Impacto para Delphos:** Ejecución remota como usuario `ubuntu`; no se probó elevación de privilegios.
- **Remediación / decisión de arquitectura:** revocación de clave conocida, bastión SSH, MFA, VLAN administrativa, registro centralizado.

### Intento 05 — Rutas no existentes

- **Fecha y hora:** 19 de octubre de 2026.
- **Fase:** 1. Reconocimiento.
- **Objetivo y puerto:** `192.168.59.10:80`.
- **Comando exacto / pasos de interfaz:** `curl -i http://192.168.59.10/config/` y `curl -i http://192.168.59.10/portal-legacy/`.
- **Resultado literal o resumen verificable:** `HTTP/1.1 404 Not Found`.
- **Conclusión:** No se confirmó exposición de esas rutas a través de esas URLs.
- **Archivo de evidencia:** registros locales `07-config.txt` y `08-portal-legacy.txt` (los 404 no tienen captura numerada independiente).
- **Impacto para Delphos:** Ninguno demostrado por esas dos respuestas.
- **Remediación / decisión de arquitectura:** Verificar de forma interna la presencia de servicios legados; no inferir su estado por `robots.txt`.


## Relación con hallazgos

| ID | Evidencia más fuerte | Estado |
|---|---|---|
| H-01 MariaDB | E-11 / E-12 | Confirmado (root DB y conteos) |
| H-02 SSH | E-14 | Confirmado (ubuntu, no root SO) |
| H-03 Backups | E-04 / E-05 | Confirmado (índice y archivos) |
| H-04 Redis | E-07 / E-13 | Confirmado (INFO) |
| H-05 Tomcat | E-08 / E-10 | Interfaz y HTTP sin TLS observados |
| H-06 Jenkins | E-09 | Panel observado; permisos administrativos sin comprobar |
| H-07 Ejemplos Tomcat | E-08 / E-10 | Detectado con QoD 99 % |
| H-08 MAC SSH | E-06 / E-10 | Detectado y enumerado |umerado |


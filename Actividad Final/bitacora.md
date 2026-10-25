# Bitácora de prueba de penetración — Delphos

**Pentester:** Alma Daniela Romero Díaz
**Fecha:** 17–24 de octubre de 2026 
**Alcance:** únicamente `192.168.59.10` | **Laboratorio:** red host-only `192.168.59.0/24`  
**Autorización:** Laboratorio aislado; carta de autorización de la práctica 
**Clasificación:** Confidencial - contiene datos sensibles ficticios.


| Fecha/hora | Fase | Comando o acción exactos | Qué se buscaba | Resultado observado | Evidencia |
|---|---|---|---|---|---|
| 18-19/oct | Preparación | `ping -c 4 192.168.59.10` | Verificar conectividad | Respuestas de Delphos, sin pérdida | captura de ping |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/` | Portada | Sitio ficticio Delphos, HTML visible y metadatos de CMS | `01-portada.html` |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/robots.txt` | Rutas publicadas | `Disallow: /backups/`, `/config/`, `/portal-legacy/` | `02-robots.txt` |
| 18-19/oct | 1. Reconocimiento | `whatweb http://192.168.59.10/` | Tecnologías | nginx 1.27.5 y CMS Delphos v1.2 | `03-whatweb.txt` |
| 18-19/oct | 1. Reconocimiento | `curl -i http://192.168.59.10/backups/` | Índice de respaldos | HTTP 200; enlaces `config.old` y `notas-migracion.txt` | `04-backups.txt` |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/backups/config.old` | Configuración expuesta | Parámetros y credenciales ficticias; **no reproducidas aquí** | captura local `config.old` |
| 18-19/oct | 1. Reconocimiento | `curl -sS http://192.168.59.10/backups/notas-migracion.txt` | Notas de migración | Coexistencia de servicios y riesgos de configuración descritos | `06-notas-migracion.txt` |
| 19/oct | 1. Reconocimiento | `curl -i http://192.168.59.10/config/` y `curl -i http://192.168.59.10/portal-legacy/` | Rutas sugeridas | Ambas respondieron 404; no afirmar que estuvieran publicadas | `07-config.txt`, `08-portal-legacy.txt` |
| 18-19/oct | 2. Escaneo | `sudo nmap -sS -sV -p- -T4 192.168.59.10 -oN evidencias/09-nmap-completo.txt` | Inventario completo TCP | Puertos abiertos 22, 80, 3306, 6379, 8080, 8083; duración 4760.29 segundos | `09-nmap-completo.txt` |
| 19/oct | 2. Escaneo | `sudo nmap -sC -sV -p 22,80,3306,6379,8080,8083 192.168.59.10 -oN evidencias/10-nmap-detalle.txt` | Enumerar versiones y scripts | OpenSSH 9.6p1, nginx 1.27.5, MariaDB 10.5.29, Redis 5.0.14, Tomcat 9.0.30 y Jetty/Jenkins | `10-nmap-detalle.txt` |
| 19/oct | 3. Enumeración | `redis-cli -h 192.168.59.10 -p 6379 PING` | Redis | `PONG` sin contraseña | `11-redis-ping.txt` |
| 19/oct | 3. Enumeración | `mariadb --disable-ssl -h 192.168.59.10 -P 3306 -u delphos_app -p` | MariaDB | Conexión de usuario de app, base `delphos` visible; sin extracción masiva | capturas MariaDB |
| 19/oct | 3. Enumeración | `curl -i http://192.168.59.10:8080/manager/html` | Tomcat Manager | HTTP 401 con Basic Auth; luego panel visible en navegador | `12-tomcat-manager.txt` + captura |
| 19/oct | 3. Enumeración | `curl -I http://192.168.59.10:8083/` | Jenkins | HTTP 200; versión 2.190.1 en cabeceras | `13-jenkins-headers.txt` |
| 24/oct | 4. OpenVAS | Target `Delphos`, Task `Pentest Delphos`, perfil `Full and fast`, OpenVAS Default | Evaluar vulnerabilidades | Task `Done`; 106 resultados brutos, 6 incluidos con filtro QoD >= 70 | `openvas-delphos.pdf` |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/mysql/mysql_version`; `set RHOSTS 192.168.59.10`; `set RPORT 3306`; `run` | MariaDB | Identificación de MariaDB 10.5.29 | `15-metasploit.txt` |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/redis/redis_server`; `set RHOSTS 192.168.59.10`; `set RPORT 6379`; `set COMMAND INFO`; `set THREADS 1`; `run` | Redis | `Found redis with INFO command`, versión 5.0.14 | `15-metasploit.txt` |
| 24/oct | 5. Metasploit | `use auxiliary/scanner/ssh/ssh_version`; `set RHOSTS 192.168.59.10`; `set RPORT 22`; `run` | SSH | OpenSSH 9.6p1; MAC `umac-64` ofrecidos | `15-metasploit.txt` |
| 24/oct | 5. Validación | `use auxiliary/scanner/mysql/mysql_login`; `run mysql://root:[CLAVE_OCULTA]@192.168.59.10:3306 createsession=true` | MariaDB | Acceso root exitoso y sesión MySQL abierta | `16-validacion-mariadb.txt` |
| 24/oct | 5. Validación | `sessions -i 1`; `query 'SELECT CURRENT_USER(), VERSION();'`; `query 'SHOW DATABASES;'` | MariaDB | `root@%`, MariaDB 10.5.29, base delphos | `16-validacion-mariadb.txt` |
| 24/oct | 5. Validación | `query 'SELECT COUNT(*) AS total_clientes FROM delphos.clientes;'`; `query 'SELECT COUNT(*) AS total_usuarios FROM delphos.usuarios_portal;'` | MariaDB | **20 clientes y 4 usuarios** (conteos, sin datos de filas) | `16-validacion-mariadb.txt` |
| 24/oct | 5. Validación | `use auxiliary/scanner/ssh/ssh_login` con `KEY_PATH=/home/kali/pentest-delphos/ssh-keys/vagrant-lab`, `set STOP_ON_SUCCESS true`, `set CreateSession true`; `run` | SSH | Autenticación reportada como exitosa; fallo al establecer shell interactiva (`Unknown platform`/`fd for nil`) | `17-validacion-ssh.txt` |
| 24/oct | 5. Validación | `ssh -i ~/pentest-delphos/ssh-keys/vagrant-lab -o IdentitiesOnly=yes -o BatchMode=yes ubuntu@192.168.59.10 'id; whoami; hostname'` | SSH | Ejecución remota como **ubuntu, uid 1001**, servidor `srv-delphos` | `18-validacion-ssh-directa.txt` |

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

- **Fecha y hora:** 19 de octubre de 2026.
- **Fase:** 3. Enumeración.
- **Objetivo y puerto:** `192.168.59.10:3306`.
- **Comando exacto / pasos de interfaz:** `mariadb -h 192.168.59.10 -P 3306 -u delphos_app -p`.
- **Resultado literal o resumen verificable:** `ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it`.
- **Conclusión:** El intento falló por incompatibilidad TLS del cliente; **no** significa que el usuario o el servicio fueran inaccesibles.
- **Archivo de evidencia:** captura de MariaDB del 19/oct.
- **Impacto para Delphos:** La configuración TLS debe corregirse de forma segura; el acceso del usuario `delphos_app` se verificó posteriormente con `--disable-ssl` dentro de la red aislada.
- **Remediación / decisión de arquitectura:** habilitar TLS correctamente, no usar `--disable-ssl` en operación real; mover la BD a VLAN de datos.

### Intento 02 — Credencial administrativa de MariaDB

- **Fecha y hora:** 24 de octubre de 2026, alrededor de 11:56.
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:3306`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/mysql/mysql_login`; `run mysql://root:[CLAVE_OCULTA]@192.168.59.10:3306 createsession=true`.
- **Resultado literal o resumen verificable:** `Success`, sesión MySQL abierta; usuario `root@%`, `delphos` visible, 20 filas de `clientes` y 4 de `usuarios_portal` contadas con `COUNT(*)`.
- **Conclusión:** Confirmado acceso administrativo **a MariaDB**; no demuestra privilegios root del SO.
- **Archivo de evidencia:** `16-validacion-mariadb.txt`.
- **Impacto para Delphos:** Riesgo de confidencialidad e integridad de registros si el servicio estuviera en producción.
- **Remediación / decisión de arquitectura:** rotar credencial, cuenta mínima, limitar 3306 a servidores autorizados.

### Intento 03 — Redis sin autenticación

- **Fecha y hora:** 24 de octubre de 2026 (hora no documentada).
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:6379`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/redis/redis_server`; `set RHOSTS 192.168.59.10`; `set RPORT 6379`; `set COMMAND INFO`; `run`.
- **Resultado literal o resumen verificable:** `Found redis with INFO command`, versión 5.0.14.
- **Conclusión:** Confirmada consulta de información sin contraseña. No se probaron comandos de escritura ni lectura de claves.
- **Archivo de evidencia:** `15-metasploit.txt`.
- **Impacto para Delphos:** Exposición de información operativa de la caché; riesgo adicional si se permitiesen otros comandos.
- **Remediación / decisión de arquitectura:** ACL de Redis, acceso desde VLAN de aplicación, firewall y TLS según despliegue.

### Intento 04 — Sesión SSH en Metasploit y validación directa

- **Fecha y hora:** 24 de octubre de 2026, alrededor de 12:18 (terminal).
- **Fase:** 5. Validación.
- **Objetivo y puerto:** `192.168.59.10:22`.
- **Comando exacto / pasos de interfaz:** `use auxiliary/scanner/ssh/ssh_login`; `set USERNAME root`; `set KEY_PATH /home/kali/pentest-delphos/ssh-keys/vagrant-lab`; `run`; `sessions -l`; `sessions -i 2`.
- **Resultado literal o resumen verificable:** El módulo informó `Success`, pero falló la creación/manipulación de sesión con `Unknown platform` y `undefined method fd for nil`. La instrucción `id` posterior se ejecutó *localmente* en Kali, por lo que no se acepta como prueba de Delphos.
- **Conclusión:** Éxito de autenticación reportado, **sin** consola interactiva funcional en Metasploit. Se validó después con `ssh -i ... ubuntu@192.168.59.10 'id; whoami; hostname'`, obteniendo `ubuntu`, `uid=1001` y `srv-delphos`.
- **Archivo de evidencia:** `17-validacion-ssh.txt` y `18-validacion-ssh-directa.txt`.
- **Impacto para Delphos:** Ejecución remota como usuario `ubuntu`; no se probó elevación de privilegios.
- **Remediación / decisión de arquitectura:** revocación de clave conocida, bastión SSH, MFA, VLAN administrativa, registro centralizado.

### Intento 05 — Rutas no existentes

- **Fecha y hora:** 19 de octubre de 2026.
- **Fase:** 1. Reconocimiento.
- **Objetivo y puerto:** `192.168.59.10:80`.
- **Comando exacto / pasos de interfaz:** `curl -i http://192.168.59.10/config/` y `curl -i http://192.168.59.10/portal-legacy/`.
- **Resultado literal o resumen verificable:** `HTTP/1.1 404 Not Found`.
- **Conclusión:** No se confirmó exposición de esas rutas a través de esas URLs.
- **Archivo de evidencia:** `07-config.txt`, `08-portal-legacy.txt`.
- **Impacto para Delphos:** Ninguno demostrado por esas dos respuestas.
- **Remediación / decisión de arquitectura:** Verificar de forma interna la presencia de servicios legados; no inferir su estado por `robots.txt`.

## Registro de cierre y protección de evidencias

- **Cierre:** se desactivó `spool` y se guardaron los archivos de evidencia de Metasploit. No se realizaron operaciones destructivas.
- **Informe automatizado:** el archivo `openvas-delphos.pdf` corresponde al reporte de la tarea completada. En la copia de entrega únicamente se ocultó el texto completo de una clave privada SSH, sin alterar la gravedad, resumen o fechas del hallazgo.
- **Resguardo:** no compartir la clave privada ni los secretos de `config.old`; el texto íntegro permanece solo en el entorno académico controlado.
- **Trazabilidad:** si el docente solicita corroborar una prueba, utilizar los archivos `evidencias/` conservados por el laboratorio. El ZIP principal contiene únicamente los tres archivos prescritos.

## Relación con hallazgos

| ID | Evidencia más fuerte | Estado |
|---|---|---|
| H-01 MariaDB | `16-validacion-mariadb.txt` | Confirmado (root DB y conteos) |
| H-02 SSH | `18-validacion-ssh-directa.txt` | Confirmado (ubuntu, no root SO) |
| H-03 Backups | `04-backups.txt`, `06-notas-migracion.txt` | Confirmado (índice y archivos) |
| H-04 Redis | `15-metasploit.txt` | Confirmado (INFO) |
| H-05 Tomcat | `12-tomcat-manager.txt`, `openvas-delphos.pdf` | Interfaz y HTTP sin TLS observados |
| H-06 Jenkins | `13-jenkins-headers.txt`, captura | Panel observado; permisos administrativos sin comprobar |
| H-07 Ejemplos Tomcat | `openvas-delphos.pdf` | Detectado con QoD 99 % |
| H-08 MAC SSH | `openvas-delphos.pdf`, `15-metasploit.txt` | Detectado y enumerado |


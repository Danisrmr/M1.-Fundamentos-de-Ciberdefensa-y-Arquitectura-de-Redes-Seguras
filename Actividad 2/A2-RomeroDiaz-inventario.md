# A.2 — Inventario de hallazgos

**Nombre:** Alma Daniela Romero Díaz  
**Fecha:** 26 de septiembre de 2026

## Banderas

| # | Fase | Técnica | Bandera | Comando exacto con el que salió |
|---|---|---|---|---|
| 1 | DNS | TXT del apex | `FLAG{nordlys_dns_txt_record_9c41}` | `dig @192.168.56.20 nordlysai.dk TXT` |
| 2 | DNS | Transferencia de zona | `FLAG{nordlys_zone_transfer_0d7e}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 3 | DNS | Espacio de nombres interno | `FLAG{nordlys_internal_zone_5a38}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 4 | DNS | Resolución inversa | `FLAG{nordlys_reverse_dns_b6f2}` | `dig @192.168.56.20 30.56.168.192.in-addr.arpa TXT` |
| 5 | Pasivo | Comentario en el código fuente | `FLAG{nordlys_source_comment_a71f}` | `curl -s http://192.168.56.10/` |
| 6 | Pasivo | `robots.txt` | `FLAG{nordlys_robots_disallow_5c93}` | `curl -s http://192.168.56.10/robots.txt` |
| 7 | Pasivo | Ruta prohibida en `robots.txt` | `FLAG{nordlys_staff_portal_2e08}` | `curl -s http://192.168.56.10/internal-tools/` |
| 8 | Pasivo | `security.txt` | `FLAG{nordlys_security_txt_9d4b}` | `curl -s http://192.168.56.10/.well-known/security.txt` |
| 9 | Pasivo | Cabecera de respuesta | `FLAG{nordlys_http_header_8b17}` | `curl -sI http://192.168.56.10/` |
| 10 | Enumeración | JavaScript del sitio | **No encontrada** | — |
| 11 | Enumeración | Copia de seguridad | `FLAG{nordlys_backup_file_3a5e}` | `curl -s http://192.168.56.10/index.html.bak` |
| 12 | Enumeración | Archivo de configuración | **No encontrada** | — |
| 13 | Enumeración | Directorio de control de versiones | **No encontrada** | — |
| 14 | Enumeración | Página no enlazada | `FLAG{nordlys_sitemap_unlinked_7e61}` | `curl -s http://192.168.56.10/careers/offer-draft-q3.html` |
| 15 | Enumeración | Listado de directorio | `FLAG{nordlys_dir_listing_b982}` | `curl -s http://192.168.56.10/uploads/backup-notes.txt` |
| 16 | Enumeración | Página de error propia | `FLAG{nordlys_custom_404_51b8}` | `curl -s http://192.168.56.10/ruta-que-no-existe` |
| 17 | Análisis | Metadatos de documento | **No encontrada** | — |
| 18 | Análisis | Exportación de datos | `FLAG{nordlys_csv_export_d13c}` | `curl -s http://192.168.56.10/uploads/medarbejderliste-eksport.csv` |
| 19 | Análisis | Respaldo del sitio | `FLAG{nordlys_old_site_archive_2b44}` | `grep -R "FLAG" nordlys-public-2024` |
| 20 | Análisis | Vhost sin registro DNS | `FLAG{nordlys_vhost_staging_af26}` | `curl -s -H "Host: dev.nordlysai.dk" http://192.168.56.10/` |

**Encontradas: 16 / 20**


## Hallazgos sin bandera

### Infraestructura — nombres de máquina, servicios internos y proveedores

La transferencia de zona reveló nombres que no deberían estar expuestos públicamente, entre ellos `backup-01.internal.nordlysai.dk`, `db-legacy-01.internal.nordlysai.dk`, `db-prod-01.internal.nordlysai.dk`, `dmz-web-01.internal.nordlysai.dk`, `git.internal.nordlysai.dk`, `mlflow.internal.nordlysai.dk`, `wiki.internal.nordlysai.dk` y `vpn-legacy.nordlysai.dk`. También se observaron `api.nordlysai.dk`, `smtp.nordlysai.dk`, `stats.nordlysai.dk` y `www.nordlysai.dk`. El TXT del dominio mostró además relación con Zenegy mediante verificación de dominio y SPF.

### Personas — datos encontrados y convenciones de nombres

En `medarbejderliste-eksport.csv` se observaron 15 registros con nombre, apellido, correo, usuario, departamento, puesto, ciudad, fecha de ingreso y teléfono.

| Patrón observado | Ejemplo |
|---|---|
| Correo | `astrid.lindqvist@nordlysai.dk` |
| Usuario del sistema | `alindqvist` |

El patrón observado para correo es `nombre.apellido@nordlysai.dk`. Para el usuario del sistema se usa la inicial del nombre seguida del apellido, sin punto.

### Credenciales o secretos encontrados

En `index.html.bak` apareció una cuenta de migración `web-migrate`, un código/contraseña representado por la bandera del laboratorio y un endpoint administrativo de desarrollo. En el respaldo `site-backup-2024.tar.gz`, el archivo `config/settings.ini` contenía un valor de `password` asociado a otra bandera. Se documentaron, pero no se probaron.

### Datos de la organización

Se observaron datos societarios y operativos como el CVR `41 92 07 65`, la dirección Ørestads Boulevard 61, 4. sal, 2300 København S, puestos de empleados, teléfonos, rangos salariales en una página de reclutamiento no enlazada, nombres de sistemas internos y proveedores externos.

## Reflexión de la Fase 1

Si la transferencia de zona estuviera correctamente restringida, las banderas del TXT del apex y de la resolución inversa podrían haberse obtenido mediante consultas directas. En cambio, la bandera asociada a la transferencia de zona y la que expone el espacio de nombres interno no habrían aparecido de esa forma. Cerrar AXFR evita entregar de una sola vez el mapa de hosts internos y reduce de forma importante la información disponible para reconocimiento.

# A.2 — Inventario de hallazgos

**Nombre:** Alma Daniela Romero Díaz  
**Fecha:** 26 de septiembre de 2026

## Banderas

Una fila por bandera. La columna del comando no es opcional: es la que demuestra que sabes cómo llegaste, y la que te sirve a ti dentro de seis meses.

| # | Fase | Técnica | Bandera | Comando exacto con el que salió |
|---|---|---|---|---|
| 1 | DNS | TXT del apex | `FLAG{nordlys_dns_txt_record_9c41}` | `dig @192.168.56.20 nordlysai.dk TXT` |
| 2 | DNS | transferencia de zona | `FLAG{nordlys_zone_transfer_0d7e}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 3 | DNS | espacio de nombres interno | `FLAG{nordlys_internal_zone_5a38}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 4 | DNS | resolución inversa | `FLAG{nordlys_reverse_dns_b6f2}` | `dig @192.168.56.20 30.56.168.192.in-addr.arpa TXT` |
| 5 | Pasivo | comentario en el código fuente | `FLAG{nordlys_source_comment_a71f}` | `curl -s http://192.168.56.10/` |
| 6 | Pasivo | `robots.txt` | `FLAG{nordlys_robots_disallow_5c93}` | `curl -s http://192.168.56.10/robots.txt` |
| 7 | Pasivo | ruta prohibida en `robots.txt` | `FLAG{nordlys_staff_portal_2e08}` | `curl -s http://192.168.56.10/internal-tools/` |
| 8 | Pasivo | `security.txt` | `FLAG{nordlys_security_txt_9d4b}` | `curl -s http://192.168.56.10/.well-known/security.txt` |
| 9 | Pasivo | cabecera de respuesta | `FLAG{nordlys_http_header_8b17}` | `curl -sI http://192.168.56.10/` |
| 10 | Enumeración | JavaScript del sitio |  |  |
| 11 | Enumeración | copia de seguridad | `FLAG{nordlys_backup_file_3a5e}` | `curl -s http://192.168.56.10/index.html.bak` |
| 12 | Enumeración | archivo de configuración |  |  |
| 13 | Enumeración | directorio de control de versiones |  |  |
| 14 | Enumeración | página no enlazada | `FLAG{nordlys_sitemap_unlinked_7e61}` | `curl -s http://192.168.56.10/careers/offer-draft-q3.html` |
| 15 | Enumeración | listado de directorio | `FLAG{nordlys_dir_listing_b982}` | `curl -s http://192.168.56.10/uploads/backup-notes.txt` |
| 16 | Enumeración | página de error propia | `FLAG{nordlys_custom_404_51b8}` | `curl -s http://192.168.56.10/ruta-que-no-existe` |
| 17 | Análisis | metadatos de documento |  |  |
| 18 | Análisis | exportación de datos | `FLAG{nordlys_csv_export_d13c}` | `curl -s http://192.168.56.10/uploads/medarbejderliste-eksport.csv` |
| 19 | Análisis | respaldo del sitio | `FLAG{nordlys_old_site_archive_2b44}` | `grep -R "FLAG" nordlys-public-2024` |
| 20 | Análisis | vhost sin registro DNS | `FLAG{nordlys_vhost_staging_af26}` | `curl -s -H "Host: dev.nordlysai.dk" http://192.168.56.10/` |

**Encontradas: 16 / 20**


## Hallazgos sin bandera

### Infraestructura — nombres de máquina, servicios internos, proveedores:

Durante la transferencia de zona pude ver varios nombres relacionados con la infraestructura de Nordlys AI. Entre ellos aparecieron servidores con nombres como `backup-01.internal.nordlysai.dk`, `db-legacy-01.internal.nordlysai.dk`, `db-prod-01.internal.nordlysai.dk`, `git.internal.nordlysai.dk`, `mlflow.internal.nordlysai.dk`, `wiki.internal.nordlysai.dk` y `vpn-legacy.nordlysai.dk`.

También aparecieron servicios como `api.nordlysai.dk`, `smtp.nordlysai.dk`, `stats.nordlysai.dk` y `www.nordlysai.dk`.

Esto me permitió observar que el DNS no solamente puede revelar la dirección de un sitio, sino también dar pistas sobre los nombres y funciones de otros equipos de la organización.

### Personas — cuántas, con qué datos, y las dos convenciones de nombres:

En el archivo `medarbejderliste-eksport.csv` encontré información de 15 empleados ficticios. Los datos incluían nombres, apellidos, correos electrónicos, usuarios, departamentos, puestos, ciudades, fechas de ingreso y teléfonos.

También pude identificar dos formas diferentes de construir los nombres:

| Patrón observado | Ejemplo |
|---|---|
| Correo | `astrid.lindqvist@nordlysai.dk` |
| Usuario del sistema | `alindqvist` |

En los correos se utiliza el patrón `nombre.apellido@nordlysai.dk`, mientras que para los usuarios se utiliza la inicial del nombre seguida del apellido. Con una lista de empleados, conocer estos patrones permitiría predecir otros correos o nombres de usuario.

### Credenciales o secretos encontrados (documentar, nunca probar):

Durante el reconocimiento encontré información sensible dentro de algunos archivos de respaldo. En `index.html.bak` apareció una referencia a una cuenta de migración llamada `web-migrate`, además de información relacionada con el entorno de desarrollo.

También encontré información dentro del respaldo `site-backup-2024.tar.gz`. Estos datos únicamente se documentaron como parte de la actividad y no se utilizaron para intentar iniciar sesión ni acceder a ningún sistema.

### Datos de la organización — societarios, financieros, proveedores:

A partir de las distintas páginas y archivos fue posible obtener información sobre la organización que normalmente no estaría reunida en un solo lugar. Por ejemplo, encontré nombres y puestos de empleados, teléfonos, direcciones de correo, nombres de sistemas internos, información relacionada con proveedores y datos contenidos en una página de reclutamiento que no estaba enlazada desde la navegación normal.

Aunque varios de estos datos por separado parecen poco importantes, juntos permiten conocer mejor la estructura y funcionamiento de la organización.

## Reflexión de la Fase 1

Si la transferencia de zona estuviera bien configurada, todavía habría podido obtener algunos datos mediante consultas directas, por ejemplo el registro TXT y parte de la información de resolución inversa. Sin embargo, no habría podido obtener de una sola vez toda la información relacionada con la zona ni los nombres internos que aparecieron en AXFR.

Por eso considero que restringir la transferencia de zona reduce bastante la cantidad de información sobre la infraestructura que una persona externa puede obtener fácilmente.

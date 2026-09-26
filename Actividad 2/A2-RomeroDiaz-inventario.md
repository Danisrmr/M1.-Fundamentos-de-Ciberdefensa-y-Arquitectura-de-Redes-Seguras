# A.2 — Inventario de hallazgos

**Nombre:** Alma Daniela Romero Díaz  
**Fecha:** 26 de septiembre de 2026

## Banderas encontradas

| # | Fase | Hallazgo | Bandera | Comando con el que la encontré |
|---|---|---|---|---|
| 1 | DNS | Registro TXT del dominio | `FLAG{nordlys_dns_txt_record_9c41}` | `dig @192.168.56.20 nordlysai.dk TXT` |
| 2 | DNS | Transferencia de zona | `FLAG{nordlys_zone_transfer_0d7e}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 3 | DNS | Nombres internos de la organización | `FLAG{nordlys_internal_zone_5a38}` | `dig @192.168.56.20 nordlysai.dk AXFR` |
| 4 | DNS | Resolución inversa | `FLAG{nordlys_reverse_dns_b6f2}` | `dig @192.168.56.20 30.56.168.192.in-addr.arpa TXT` |
| 5 | Reconocimiento pasivo | Comentario dentro del código fuente | `FLAG{nordlys_source_comment_a71f}` | `curl -s http://192.168.56.10/` |
| 6 | Reconocimiento pasivo | Información en `robots.txt` | `FLAG{nordlys_robots_disallow_5c93}` | `curl -s http://192.168.56.10/robots.txt` |
| 7 | Reconocimiento pasivo | Portal interno encontrado por una ruta publicada | `FLAG{nordlys_staff_portal_2e08}` | `curl -s http://192.168.56.10/internal-tools/` |
| 8 | Reconocimiento pasivo | Archivo `security.txt` | `FLAG{nordlys_security_txt_9d4b}` | `curl -s http://192.168.56.10/.well-known/security.txt` |
| 9 | Reconocimiento pasivo | Información en las cabeceras HTTP | `FLAG{nordlys_http_header_8b17}` | `curl -sI http://192.168.56.10/` |
| 10 | Enumeración | JavaScript del sitio | **No encontrada** | — |
| 11 | Enumeración | Copia de seguridad de una página | `FLAG{nordlys_backup_file_3a5e}` | `curl -s http://192.168.56.10/index.html.bak` |
| 12 | Enumeración | Archivo de configuración | **No encontrada** | — |
| 13 | Enumeración | Directorio de control de versiones | **No encontrada** | — |
| 14 | Enumeración | Página que no estaba enlazada desde el sitio | `FLAG{nordlys_sitemap_unlinked_7e61}` | `curl -s http://192.168.56.10/careers/offer-draft-q3.html` |
| 15 | Enumeración | Información localizada dentro de los archivos publicados | `FLAG{nordlys_dir_listing_b982}` | `curl -s http://192.168.56.10/uploads/backup-notes.txt` |
| 16 | Enumeración | Página de error personalizada | `FLAG{nordlys_custom_404_51b8}` | `curl -s http://192.168.56.10/ruta-que-no-existe` |
| 17 | Análisis de archivos | Metadatos del documento PDF | **No encontrada** | — |
| 18 | Análisis de archivos | Exportación de información de empleados | `FLAG{nordlys_csv_export_d13c}` | `curl -s http://192.168.56.10/uploads/medarbejderliste-eksport.csv` |
| 19 | Análisis de archivos | Información dentro de un respaldo antiguo | `FLAG{nordlys_old_site_archive_2b44}` | `grep -R "FLAG" nordlys-public-2024` |
| 20 | Enumeración | Sitio virtual sin registro DNS | `FLAG{nordlys_vhost_staging_af26}` | `curl -s -H "Host: dev.nordlysai.dk" http://192.168.56.10/` |

**Total de banderas encontradas: 16 de 20.**

---

## Otros hallazgos

### Infraestructura encontrada por DNS

La transferencia de zona fue uno de los comandos que más información me mostró. Además de los nombres públicos, aparecieron nombres relacionados con equipos y servicios internos de Nordlys AI.

Entre los que observé estaban:

- `backup-01.internal.nordlysai.dk`
- `db-legacy-01.internal.nordlysai.dk`
- `db-prod-01.internal.nordlysai.dk`
- `dmz-web-01.internal.nordlysai.dk`
- `git.internal.nordlysai.dk`
- `mlflow.internal.nordlysai.dk`
- `wiki.internal.nordlysai.dk`
- `vpn-legacy.nordlysai.dk`

También encontré otros nombres como:

- `api.nordlysai.dk`
- `smtp.nordlysai.dk`
- `stats.nordlysai.dk`
- `www.nordlysai.dk`

Esto me permitió entender que una transferencia de zona abierta puede mostrar bastante información sobre cómo está organizada una red, incluso sin intentar entrar en ningún sistema.

---

### Información de empleados

Dentro del archivo:

`medarbejderliste-eksport.csv`

encontré información de 15 empleados ficticios.

El archivo incluía datos como:

- Nombre
- Apellido
- Correo
- Usuario
- Departamento
- Puesto
- Ciudad
- Fecha de ingreso
- Teléfono

También pude identificar dos convenciones de nombres diferentes.

Ejemplo de correo:

`astrid.lindqvist@nordlysai.dk`

Ejemplo de usuario:

`alindqvist`

El correo sigue aproximadamente el patrón:

`nombre.apellido@nordlysai.dk`

Mientras que el nombre de usuario utiliza:

`inicial del nombre + apellido`

Esto es importante porque, teniendo una lista de empleados, sería relativamente sencillo intentar predecir otros nombres de usuario o correos de la organización.

---

### Archivos de respaldo y datos sensibles

Uno de los hallazgos que más llamó mi atención fue que había archivos antiguos y copias de seguridad accesibles desde el servidor web.

En `index.html.bak` encontré información relacionada con una cuenta de migración llamada:

`web-migrate`

Además, había referencias al entorno de desarrollo y otra información que no debería encontrarse dentro de un archivo publicado.

También descargué el respaldo:

`site-backup-2024.tar.gz`

y, después de extraerlo, revisé su contenido.

Para buscar las banderas dentro del respaldo utilicé:

`grep -R "FLAG" nordlys-public-2024`

Esto demostró que aunque un respaldo sea antiguo, todavía puede contener configuraciones o información que siga siendo útil para conocer la infraestructura de una organización.

Durante toda la actividad únicamente documenté esta información. No probé ninguna de las credenciales encontradas.

---

### Información de la organización

Además de la información técnica, también encontré datos sobre la empresa y sus empleados.

Entre ellos aparecieron:

- Nombres de empleados
- Puestos de trabajo
- Correos
- Teléfonos
- Información de sistemas internos
- Rangos salariales dentro de una página de reclutamiento
- Datos de proveedores externos
- Nombres de servidores
- Información relacionada con el entorno de desarrollo

Por separado, algunos datos pueden parecer poco importantes, pero al reunirlos se puede construir una imagen bastante completa de la organización.

---

## Reflexión sobre la transferencia de zona

La transferencia de zona fue uno de los hallazgos que más información mostró porque con un solo comando pude conocer varios nombres relacionados con la infraestructura de Nordlys AI.

Si AXFR estuviera correctamente restringido, todavía sería posible consultar algunos registros de manera individual, por ejemplo el TXT del dominio o realizar algunas consultas inversas. Sin embargo, no habría podido obtener tan fácilmente una lista completa de nombres internos.

Por eso considero que restringir la transferencia de zona disminuye bastante la información que una persona externa puede conocer sobre la red.

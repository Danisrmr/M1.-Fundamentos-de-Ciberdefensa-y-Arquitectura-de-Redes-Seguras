# Reporte de Superficie Expuesta — Nordlys AI ApS

**Analista:** Alma Daniela Romero Díaz  
**Fecha:** 26 de septiembre de 2026  
**Alcance:** dominio `nordlysai.dk` y los servicios que resuelve, en entorno de laboratorio autorizado. Sólo reconocimiento: no se explotó ni se autenticó nada.

---

## 1 · Resumen para dirección

Durante el reconocimiento encontré que Nordlys AI tiene publicada más información de la que realmente necesita mostrar. Fue posible encontrar datos de empleados, nombres de sistemas internos, archivos de respaldo y páginas que no estaban enlazadas desde el sitio principal. Aunque cada uno de estos datos por separado puede parecer poco importante, al reunirlos se obtiene bastante información sobre cómo funciona la organización. Esto podría facilitar ataques dirigidos contra empleados o ayudar a identificar sistemas de interés. Considero necesario revisar qué archivos e información se mantienen públicos y retirar aquellos que no sean necesarios.

## 2 · Hallazgos

Ordenados por riesgo, no por el orden en que los encontraste.

| # | Hallazgo | Cómo se encontró | Riesgo | Por qué importa |
|---|---|---|---|---|
| 1 | Archivo con información de empleados | Se encontró una exportación CSV dentro de los archivos públicos del sitio | Alto | Contiene nombres, correos, usuarios, puestos y teléfonos que podrían utilizarse para preparar ataques dirigidos o suplantar a empleados. |
| 2 | Archivos de respaldo con información sensible | Se encontraron copias antiguas del sitio y archivos de respaldo durante la enumeración | Alto | Un respaldo puede conservar usuarios, configuraciones o información que ya no es visible en la versión actual del sitio. |
| 3 | Transferencia de zona DNS abierta | Se realizó una transferencia de zona sobre `nordlysai.dk` | Medio | Permitió conocer rápidamente varios nombres de servidores y sistemas relacionados con la organización. |
| 4 | Portal de herramientas internas accesible | La ruta apareció al revisar `robots.txt` | Medio | Aunque la página no estaba enlazada desde el menú principal, fue posible descubrirla y obtener información sobre herramientas internas. |
| 5 | Sitio de desarrollo sin registro DNS | Se encontró al probar el nombre virtual `dev.nordlysai.dk` | Medio | Demuestra que un sitio puede seguir disponible en el servidor aunque no aparezca en los registros DNS. |
| 6 | Comentarios con información de desarrollo | Se revisó el código fuente de la página principal | Medio | Los comentarios mostraban datos relacionados con el entorno de desarrollo que no eran necesarios para los visitantes. |
| 7 | Página de error con información técnica | Se solicitó una ruta inexistente | Medio | La respuesta mostraba datos del servidor y de depuración que podían dar pistas sobre su configuración. |
| 8 | Página no enlazada públicamente | Se localizó una página de reclutamiento que no aparecía en la navegación normal | Medio | Incluía información de la organización que probablemente no estaba destinada a ser encontrada de esa manera. |
| 9 | Información técnica en las cabeceras HTTP | Se revisaron las cabeceras de respuesta del servidor | Bajo | Revela datos sobre el servidor y la compilación que ayudan a conocer mejor la tecnología utilizada. |
| 10 | `robots.txt` revela rutas interesantes | Se consultó directamente el archivo `robots.txt` | Bajo | En lugar de proteger contenido, permitió identificar rutas que después podían revisarse directamente. |

## 3 · Las cinco correcciones más urgentes

1. Restringir la transferencia de zona DNS para que solamente los servidores autorizados puedan realizarla.
2. Retirar del servidor público los respaldos, copias antiguas y archivos que contienen información de empleados.
3. Revisar el proceso de publicación para evitar que archivos como `.bak`, configuraciones o respaldos terminen en el servidor.
4. Restringir el acceso a las herramientas internas y al entorno de desarrollo para que no puedan consultarse públicamente.
5. Revisar los comentarios del código, las cabeceras y las páginas de error para evitar mostrar información técnica innecesaria.

## 4 · Sobre la transferencia de zona

La transferencia de zona me permitió obtener en una sola consulta varios nombres relacionados con la infraestructura de Nordlys AI. Sin ella habría tenido que buscar los registros de manera individual y probablemente no habría encontrado algunos de los nombres internos. Al administrador le recomendaría limitar AXFR solamente a los servidores DNS que realmente necesiten realizar la transferencia. De esta forma se evita entregar fácilmente un mapa de los nombres utilizados dentro de la organización.

## 5 · Confrontación con las notas del administrador

Se rellena al final, después de leer `/uploads/backup-notes.txt`.

| | |
|---|---|
| **Ellos lo sabían, yo no lo encontré** | En las notas aparecían referencias a elementos como un archivo `.env`, un directorio `.git` y un documento que debía revisarse por sus metadatos. Durante mi reconocimiento no logré encontrar las banderas correspondientes a esos elementos. |
| **Yo lo encontré, ellos no lo tenían apuntado** | Encontré información adicional como la transferencia de zona DNS, nombres internos, información mostrada en la página de error, datos en las cabeceras del servidor y un sitio de desarrollo que respondía al utilizar el nombre virtual correspondiente. |

Al comparar los resultados me llamó la atención que algunos problemas ya eran conocidos, pero todavía seguían presentes. Esto muestra que detectar un problema no es suficiente si después no se le da seguimiento. Varias de las exposiciones parecían provenir de archivos o configuraciones que alguna vez fueron útiles y simplemente quedaron publicadas. Por eso también hace falta revisar periódicamente qué información continúa disponible.

## 6 · Datos personales recogidos en este ejercicio

Durante la actividad encontré un archivo con información de 15 empleados ficticios que incluía nombres, correos, usuarios, puestos, ciudades y teléfonos. Si fueran datos reales, recogería únicamente la información necesaria para el análisis y evitaría conservar copias adicionales. También guardaría los archivos de manera protegida mientras realizara el trabajo y evitaría colocar los datos personales completos en el reporte final. Una vez terminado el análisis, eliminaría las copias que ya no fueran necesarias.

## 7 · Pregunta guía

El DNS permite conocer principalmente nombres, direcciones y algunos servicios relacionados con la organización. En cambio, al enumerar el servidor web se pueden encontrar páginas, archivos, respaldos y otros contenidos que no necesariamente aparecen en DNS. Por eso ambos tipos de reconocimiento muestran información diferente y se complementan.

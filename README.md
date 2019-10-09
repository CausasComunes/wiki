# mediawiki-der

### Infraestructura general de servicios
Este software enastá basado en [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki/es). Por ende, cuenta con un servidor web, Apache + PHP **7.2.19**, y una base de datos MySql **5.7**. Además, al utilizar la extensión [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor) para habilitar un editor visual tipo *office* en los artículos, también requiere de un servidor Node.js con [Parsoid](https://www.mediawiki.org/wiki/Parsoid). Esto resulta sumamente simple de implementar utilizando Docker. Afortunadamente MediaWiki tiene una [imagen oficial en DockerHub](https://hub.docker.com/_/mediawiki). Debido a compatibilidades con las extensiones que utilizamos, usamos la versión **1.32.2** de MediaWiki, como se puede apreciar en nuestro [`Dockerfile`](Dockerfile). Para Parsoid utilizamos la imagen [thenets/parsoid](https://hub.docker.com/r/thenets/parsoid/) versión **0.10**.

### Configuración de MediaWiki
Si bien la aplicación de MediaWiki está en PHP, es muy poco lo que se programa en este lenguage. Donde más se usa es en el archivo de configuración [`LocalSettings.php`](LocalSettings.php). Este archivo se genera automáticamente al utilizar el asistente de instalación de MediaWiki. Posteriormente le agregamos configuración personalizada, a partir de la línea *142* del mismo archivo. Las extensiones que vienen instaladas en esta MediaWiki son:
- [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor): editor tipo *office* para los artículos
- [TemplateStyles](https://www.mediawiki.org/wiki/Extension:TemplateStyles): permite estilos css incrustados dentro de las [plantillas](https://www.mediawiki.org/wiki/Help:Templates) (que son básicamente componentes html reutilizables)
- [Flow](https://www.mediawiki.org/wiki/Extension:Flow): para tener páginas de discusión más amigables, tipo foro.
- [Echo](https://www.mediawiki.org/wiki/Extension:Echo): una dependencia de Flow.
- [EditNotify](https://www.mediawiki.org/wiki/Extension:EditNotify): para notificar por mensaje interno a otrxs usuarixs cuando se hacen modificaciones en artículos.

Todas estas extensiones tienen sus propias instrucciones de configuración dentro de `LocalSettings.php`, y algunas unos comandos dentro del `Dockerfile`. Si desean utilizar más extensiones, recuerden siempre utilizar las versiones **1.32** de las mismas.

### Pruebas preliminares
Para probar MediaWiki en su estado más puro podemos levantar un contenedor de Docker haciendo:   
`docker run --name prueba-mediawiki -p 8080:80 mediawiki:1.32`   
y navegar a [http://localhost:8080/](http://localhost:8080/) (van a tener que elegir SQLite como base de datos).

Una vez completado el asistente de instalación deben bajarse el archivo `LocalSetting.php` generado y copiarlo al contenedor. Por ejemplo si el archivo se descargó en `~/Downloads/`, haríamos:   
`docker cp ~/Downloads/LocalSettings.php prueba-mediawiki:/var/www/html/`

### Prueba con base de datos volátil
Debido a que el contenedor de MediaWiki al levantarse busca instantáneamente conectarse a la base de datos y esta requiere un mínimo de configuración antes de ser utilizada, nos resultó más fácil separar el contenedor de MySql del resto. De esta forma, al correr el contenedor de MediaWiki, la base de datos ya esta lista para recibir la conexión.

Para probar este sistema con una base de datos volátil pueden crear un contenedor de MySql:   
`docker run --name mysql --network red-mediawiki -eMYSQL_ROOT_PASSWORD=iF8D53Lovy6Js3 mysql:5.7`

Después conectarse a la consola `mysql` del contenedor:   
`docker exec -it mysql mysql -piF8D53Lovy6Js3`

Crear la base de datos y unx usuarix para la misma:   
`CREATE DATABASE mediawikidb;`   
`GRANT ALL PRIVILEGES ON mediawikidb.* TO 'mediawiki747895'@'%' IDENTIFIED BY 'kjavjkKL2949JDFJK757ysd82487';`

Y posteriormente levantar el Compose: `docker-compose up`

Si toda va bien, deberían poder acceder a la wiki vía [http://localhost:8020/](http://localhost:8020/).

La mayoría de los parámetros usados en estos comandos tienen que corresponderse con las variables de entorno definidas en [`docker-compose.yml`](docker-compose.yml)

-----
-----
-----
*docu vieja*


Para usar simplemente hacer `docker-compose up` dentro de la carpeta del proyecto.

Asegurarse de tener el puerto `8080` libre o cambiar la configuración de `ports` y la variable de entorno `MW_SITE_SERVER` en `docker-compose.yml`. Para cambiar la ip/dominio de la wiki es necesario rebuildear.

Credenciales por defecto del admin: `admin` `123123123`.


Si en algún momento desean debugear un error agregar la siguiente línea a `LocalSettings.php` del contenedor:
`$wgShowExceptionDetails = true; $wgDebugToolbar = true; $wgShowDebug = true; $wgDevelopmentWarnings = true; $wgDebugLogFile = "/tmp/{$wgSitename}-debug.log";`

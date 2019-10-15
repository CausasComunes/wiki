### Infraestructura general de servicios
Este software está basado en [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki/es). Por ende, cuenta con un servidor web, Apache + PHP **7.2.19**, y una base de datos MySql **5.7**. Además, al utilizar la extensión [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor) para habilitar un editor visual tipo *office* en los artículos, también requiere de un servidor Node.js con [Parsoid](https://www.mediawiki.org/wiki/Parsoid). Esto resulta sumamente simple de implementar utilizando Docker. Afortunadamente MediaWiki tiene una [imagen oficial en DockerHub](https://hub.docker.com/_/mediawiki). Debido a compatibilidades con las extensiones que utilizamos, usamos la versión **1.32.2** de MediaWiki, como se puede apreciar en nuestro [`Dockerfile`](Dockerfile). Para Parsoid utilizamos la imagen [thenets/parsoid](https://hub.docker.com/r/thenets/parsoid/) versión **0.10**.

### Configuración de MediaWiki
Si bien la aplicación de MediaWiki está en PHP, es muy poco lo que se programa en este lenguage. Donde más se usa es en el archivo de configuración [`LocalSettings.php`](LocalSettings.php). Este archivo se genera automáticamente al utilizar el asistente de instalación de MediaWiki la primera vez que se corre. Posteriormente, le hemos agregado configuración personalizada, a partir de la línea *142* del mismo archivo. Las extensiones que vienen instaladas en nuestra MediaWiki son:
- [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor): editor tipo *office* para los artículos
- [TemplateStyles](https://www.mediawiki.org/wiki/Extension:TemplateStyles): permite estilos css incrustados dentro de las [plantillas](https://www.mediawiki.org/wiki/Help:Templates) (que son básicamente componentes html reutilizables)
- [Flow](https://www.mediawiki.org/wiki/Extension:Flow): para tener páginas de discusión más amigables, estilo foro.
- [Echo](https://www.mediawiki.org/wiki/Extension:Echo): una dependencia de Flow.
- [EditNotify](https://www.mediawiki.org/wiki/Extension:EditNotify): para notificar por mensaje interno a otrxs usuarixs cuando se hacen modificaciones en artículos.

Todas estas extensiones tienen sus propias instrucciones de configuración dentro de `LocalSettings.php`, y algunas unos comandos dentro del `Dockerfile`. Si desean utilizar más extensiones, recuerden siempre utilizar las versiones **1.32** de las mismas.

### Prueba preliminar
Para probar MediaWiki en su estado más puro podemos levantar un contenedor de Docker haciendo:   
`docker run --name prueba-mediawiki -p 8080:80 mediawiki:1.32`   
y navegar a [http://localhost:8080/](http://localhost:8080/) (van a tener que elegir SQLite como base de datos en el asistente de instalación).

Una vez completada la instalación deben bajarse el archivo `LocalSetting.php` generado y copiarlo al contenedor. Por ejemplo si el archivo se descargó en `~/Downloads/`, haríamos:   
`docker cp ~/Downloads/LocalSettings.php prueba-mediawiki:/var/www/html/`

### Prueba de nuestra MediaWiki con base de datos volátil
Debido a que el contenedor de MediaWiki al levantarse busca instantáneamente conectarse a la base de datos y esta requiere un mínimo de configuración antes de ser utilizada, nos resultó más fácil separar el contenedor de MySql del resto. De esta forma, al correr el contenedor de MediaWiki, la base de datos ya esta lista para recibir la conexión.

Para probar nuestra MediaWiki con una base de datos volátil deben crear una red local de Docker y un contenedor de MySql:   
```
docker network create red-wiki-prueba
docker run --name mysql-mediawiki --network red-wiki-prueba -eMYSQL_ROOT_PASSWORD=N44PvQGLbQqCW4ddq8Ng mysql:5.7
```

Después conectarse a la consola `mysql` del contenedor:   
`docker exec -it mysql-mediawiki mysql -pN44PvQGLbQqCW4ddq8Ng`

Crear la base de datos y unx usuarix para la misma:   
```
CREATE DATABASE mediawikidb;
GRANT ALL PRIVILEGES ON mediawikidb.* TO '6uJ5bkUE6cTZNU2LEbAw'@'%' IDENTIFIED BY 'GveN36tyUuRTG9XSyhwJ';
```

Y posteriormente levantar el Compose: `docker-compose up`

Si toda va bien, deberían poder acceder a la wiki vía [http://localhost:8020/](http://localhost:8020/), y acceder a la cuenta de administración con las credenciales `admin`/`tTBjPXMMUzu9c3enjZqM`

(la mayoría de los parámetros usados en estos comandos tienen que corresponderse con las variables de entorno definidas en [`docker-compose.yml`](docker-compose.yml))

Finalmente deben cargar los estilos personalizados. Para esto hay que ir a [index.php/Especial:Importar](http://localhost:8020/index.php/Especial:Importar), seleccionar el archivo [`Common-css.xml`](Common-css.xml), poner "mw" en el campo "Prefijo de interwiki" e importar. Y listo!

### Algunas configuraciones útiles

#### Volúmenes del contenedor de MediaWiki
La MediaWiki almacena las imágenes subidas por lxs usuarixs en la carpeta `/var/www/html/images`, con lo cual convendría hacerle un volumen a dicho directorio para no perder las imágenes en caso de rebuildeo/redeploy del contenedor. Los artículos, su contenido, lxs usuarixs registradxs y demás datos se guardan en la BBDD.

#### Activar modo debug (reporte de errores)

Si en algún momento desean debugear un error, agregar la ssiguientes líneas a `LocalSettings.php` (pueden conectarse directo a la consola del contenedor y editar el archivo in situ con el editor `nano`):
```$wgShowExceptionDetails = true;
$wgDebugToolbar = true;
$wgShowDebug = true;
$wgDevelopmentWarnings = true;
$wgDebugLogFile = "/tmp/{$wgSitename}-debug.log";
```

#### Agregar estilos (css) globales
Editar el archivo [index.php/MediaWiki:Common.css](http://localhost:8020/index.php/MediaWiki:Common.css)

#### Editar panel de navegación
Para editar el panel lateral de "Navegación" hay que entrar a la página [index.php?title=MediaWiki:Sidebar&action=edit](http://localhost:8020/index.php?title=MediaWiki:Sidebar&action=edit)

#### Tutoriales oficiales
Algunas configuraciones comunes: [mediawiki.org/wiki/Manual:LocalSettings.php](https://www.mediawiki.org/wiki/Manual:LocalSettings.php)   
Listado completo de configuraciones: [mediawiki.org/wiki/Manual:Configuration_settings](https://www.mediawiki.org/wiki/Manual:Configuration_settings)   
Tutoriales sobre uso y funcionamiento de MediaWiki: [mediawiki.org/wiki/Help:Contents](https://www.mediawiki.org/wiki/Help:Contents)   

### General service structure
This software is based on [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki/es). Thus, it has an Apache + PHP **7.2.19** web server and a MySql **5.7** database.
It also uses the [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor) extension to enable a visual editor, like *office* ones, for its articles. This requires a Node.js server with [Parsoid](https://www.mediawiki.org/wiki/Parsoid). All this is an easy task if you use Docker. Fortunately,  MediaWiki has an [official DockerHub image](https://hub.docker.com/_/mediawiki). Due to extension compatibilities we use version **1.32.2** of it, as you can see in the [`Dockerfile`](Dockerfile). For the Parsoid server we use [thenets/parsoid](https://hub.docker.com/r/thenets/parsoid/) version **0.10**.

### Configuring MediaWiki
Thou MediaWiki is written in PHP, very little knowledge of it is required to be able to configure or modify MediaWiki.  Where PHP is most used is in [`LocalSettings.php`](LocalSettings.php) configuration file. This file is created automatically the first time MediaWiki is installed, after you complete the installation wizard. We have added some custom configuration, starting from line *142* of the file just mentioned. The extensions we have added to our MediaWiki are:
- [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor): *office*-like article editor
- [TemplateStyles](https://www.mediawiki.org/wiki/Extension:TemplateStyles): gives the possibility to add css styles inside [templates](https://www.mediawiki.org/wiki/Help:Templates) (which basically are reusable html snippets),
- [Flow](https://www.mediawiki.org/wiki/Extension:Flow): for customizing discussion pages, making it more user friendly and modern.
- [Echo](https://www.mediawiki.org/wiki/Extension:Echo): a Flow dependency.
- [EditNotify](https://www.mediawiki.org/wiki/Extension:EditNotify): an internal notification system to warn admin users when an article is edited.

All this extensions are installed and configured using some commands inside `LocalSettings.php` and `Dockerfile`. If you wish to try some other extension, always remember to use their **1.32** version.

### Preliminary test
To test the original MediaWiki with its default configuration, you can start a Docker container by doing:
`docker run --name prueba-mediawiki -p 8080:80 mediawiki:1.32`   
and navigate to  [http://localhost:8080/](http://localhost:8080/) (you should select SQLite as database in the installation wizard)

Once you complete the installation download the `LocalSetting.php` generated and copy it inside the container. If the file was downloaded at `~/Downloads/`, we would do:
`docker cp ~/Downloads/LocalSettings.php prueba-mediawiki:/var/www/html/`

### Testing our MediaWiki using an ephemeral database
Upon start, MediaWiki container searches instantly for its database and tries to connect with it. Due to this behavior, its difficult to configure a MySql database that is ready for this connection in the same docker-compose as the wiki. We’ve solved this separating these services, and first configuring the MySql and just then, starting the wiki. This way, the database container is always ready to receive the connection.

To test out MediaWiki using an ephemeral database, first you should create a local Docker network and a MySql container:
```
docker network create red-wiki-prueba
docker run --name mysql-mediawiki --network red-wiki-prueba -eMYSQL_ROOT_PASSWORD=N44PvQGLbQqCW4ddq8Ng mysql:5.7
```

Afterwards, log into the container’s `mysql` console:
`docker exec -it mysql-mediawiki mysql -pN44PvQGLbQqCW4ddq8Ng`

Create the database and an admin user for it:
```
CREATE DATABASE mediawikidb;
GRANT ALL PRIVILEGES ON mediawikidb.* TO '6uJ5bkUE6cTZNU2LEbAw'@'%' IDENTIFIED BY 'GveN36tyUuRTG9XSyhwJ';
```

Then, run Compose: `docker-compose up`

If everything went OK you should be able to see the wiki on [http://localhost:8020/](http://localhost:8020/), and login with `admin`/`tTBjPXMMUzu9c3enjZqM`.

(most of the parameters used in this commands are set as environment variables in [`docker-compose.yml`](docker-compose.yml))

Finally, load the custom css styles. Go to [index.php/Especial:Importar](http://localhost:8020/index.php/Especial:Importar), select the file [`Common-css.xml`](Common-css.xml), input “mw” in the "Prefijo de interwiki" field and import. We are done!

### Some useful configurations

#### Volumes inside MediaWiki container
MediaWiki saves uploaded images in the folder `/var/www/html/images`. We have to create a volume in that directory to preserve uploaded user images between container’s rebuilds/deploys. The articles, user data, and all other data is saved in the DB.

#### Activating debug mode (with detailed error reporting)

If you wish to debug some error in the application, add the following lines in `LocalSettings.php` (you can edit directly this file connecting to the container’s console and using the `nano` editor):
```$wgShowExceptionDetails = true;
$wgDebugToolbar = true;
$wgShowDebug = true;
$wgDevelopmentWarnings = true;
$wgDebugLogFile = "/tmp/{$wgSitename}-debug.log";
```

#### Adding global css styles

Just edit [index.php/MediaWiki:Common.css](http://localhost:8020/index.php/MediaWiki:Common.css).

#### Changing the navigation panel
For this, go to the page [index.php?title=MediaWiki:Sidebar&action=edit](http://localhost:8020/index.php?title=MediaWiki:Sidebar&action=edit).

#### Official tutorials
Some common MediaWiki configuration: [mediawiki.org/wiki/Manual:LocalSettings.php](https://www.mediawiki.org/wiki/Manual:LocalSettings.php)   
Complete configuration list: [mediawiki.org/wiki/Manual:Configuration_settings](https://www.mediawiki.org/wiki/Manual:Configuration_settings)   
MediaWiki guides: [mediawiki.org/wiki/Help:Contents](https://www.mediawiki.org/wiki/Help:Contents)   

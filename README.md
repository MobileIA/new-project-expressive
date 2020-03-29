# Nuevo proyecto para APIs con Zend Expressive

1. Crear proyecto
```bash
composer create-project zendframework/zend-expressive-skeleton name_of_project
```
2. Activar configuración personalizada:
- Modular
- Zend Service Manager
- Fast Route
- None template
- Whoops
3. Agregar repositorios en el composer.json:
```js
"repositories": [{
            "type": "git",
            "url": "https://github.com/MobileIA/mia-eloquent-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-core-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/authentication.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-authentication-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-firebase-zf3.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-firebase-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-mail-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-newsletter-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-slider-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-metatags-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-job-expressive.git"
        },
        {
            "type": "git",
            "url": "https://github.com/MobileIA/mia-installer-expressive.git"
        }
    ]
```
3. Configuración de la base de datos:
```bash
composer require illuminate/database
composer require mobileia/mia-core-expressive
composer require mobileia/mia-eloquent-expressive
```
4. Crear archivo "eloquent.global.php" en "config/autoload":
```php
return [
    'eloquent' => [
        'driver'    => 'mysql',
        'host'      => 'localhost:8889',
        'database'  => 'expressive_test',
        'username'  => 'root',
        'password'  => 'root',
        'charset'   => 'utf8',
        'collation' => 'utf8_general_ci',
    ]
];
```
5. Iniciar base de datos desde "index.php":
```php
/** @var \Psr\Container\ContainerInterface $container */
$container = require 'config/container.php';

// Inicializar Database
Mobileia\Expressive\Database\Eloquent::install($container);
```
6. Activar para recibir JSON, abrir archivo "config/pipeline.php":
```php
use Zend\Expressive\Helper\BodyParams\BodyParamsMiddleware;
//...
//$app->pipe(ServerUrlMiddleware::class);
$app->pipe(BodyParamsMiddleware::class);
```
7. Crear primer modelo:
```php
namespace App\Model;

/**
 * Description of Ranking
 *
 * @author matiascamiletti
 */
class Ranking extends \Illuminate\Database\Eloquent\Model
{
    protected $table = 'ranking';
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    //public $timestamps = false;
}
```
8. Incluir libreria de Autenticación: https://github.com/MobileIA/mia-authentication-expressive
9. Generar Primer servicio:
- Crear Archivo: src/App/src/Handler/FetchHandler y extendemos de: \Mobileia\Expressive\Auth\Request\MiaAuthRequestHandler
```php
class FetchHandler extends \Mobileia\Expressive\Request\MiaRequestHandler
{
    /**
     * Servicio para obtener los datos
     * @param \Psr\Http\Message\ServerRequestInterface $request
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function handle(\Psr\Http\Message\ServerRequestInterface $request): \Psr\Http\Message\ResponseInterface 
    {
        // Obtener usuario
        $user = $this->getUser($request);
        // Obtener parametro de Actualización
        $updatedAt = $this->getParam($request, 'updated_at', null);
        // Obtenemos registros
        $activities = \App\Repository\ActivityRepository::fetchAll($user->id, $updatedAt);
        // Devolvemos respuesta
        return new \Mobileia\Expressive\Diactoros\MiaJsonResponse($activities->toArray());
    }
}
```
10. Incluir ruta en: /config/routes.php:
```php
    /**
     * Servicio para enviar la puntuación del jugador
     */
    $app->route('/ranking/send', [
        // Validar los parametros requeridos
        new Mobileia\Expressive\Request\MiaVerifyParamHandler(array('name', 'game_id', 'points')),
        // Verificar que sea un usuario logueado
        \Mobileia\Expressive\Auth\Handler\AuthHandler::class,
        App\Handler\SendRankingHandler::class], ['GET', 'POST'], 'ranking.send');
```
11. Activar CORS:
```bash
composer require tuupola/cors-middleware
```
```php
//$app->pipe(CorsMiddleware::class);
$app->pipe(new \Tuupola\Middleware\CorsMiddleware([
    "origin" => ["*"],
    "methods" => ["GET", "POST", "PUT", "PATCH", "DELETE"],
    "headers.allow" => ["Authorization", "If-Match", "If-Unmodified-Since", "Content-Type", "Accept", "Origin", "X-Auth-Token", "X-Requested-With", "Access-Control-Allow-Headers", "Access-Control-Allow-Origin"],
    "headers.expose" => ["Etag"],
    "credentials" => false,
    "cache" => 0,
]));
```

## Activar StackDriver en App Engine
1. Agregar en el archivo config/autoload/zend-expressive.global.php
```php
'dependencies' => [
    'invokables' => [
    ],
    'factories'  => [
        \Zend\Expressive\Middleware\ErrorResponseGenerator::class       => \Mobileia\Expressive\Middleware\StackDriverErrorMiddleware::class
    ],
],
```
2. Agregar librerias:
```js
"google/cloud-error-reporting": "^0.16.0",
        "google/cloud-logging": "^1.19",
        "google/cloud-storage": "^1.18"
```

## Activar modo desarrollador
```bash
$ composer development-enable  # enable development mode
$ composer development-disable # disable development mode
$ composer development-status  # show development status
```

## Como iniciar servidor para testear
```bash
composer run --timeout=0 serve
```

## Testing
PHPUnit y PHP_CodeSniffer son instalados por default. Para ejecutar los tests:
```bash
composer check
```

## Producción
Como publicar en el servidor.
1. Clonar repositorio:
```bash
git clone https://matiascamiletti@bitbucket.org/matiascamiletti/juegos-api.git
```
2. Instalar dependencias: 
```bash
sudo composer install --no-dev
sudo composer update --no-dev
```
3. Dar permisos a la carpeta de cache:
```bash
sudo chmod -R 777 data/cache/
```

- Limpiar cache
```bash
composer clear-config-cache
```

- Configurar Apache, Crear archivo host.domain.com.conf en "/etc/apache2/sites-available"
```txt
<VirtualHost *:80>
    ServerAdmin matiascamiletti@mobileia.com
    ServerName api.iagram.mobileia.com
    ServerAlias www.api.iagram.mobileia.com
    DocumentRoot /var/www/ia-gram-api/public
    ErrorLog ${APACHE_LOG_DIR}/ia-gram-api.error.log
    CustomLog ${APACHE_LOG_DIR}/ia-gram-api.access.log combined
    AccessFileName .htaccess
       RewriteEngine on
       <Directory /var/www/ia-gram-api/public >
              AllowOverride All
       </Directory>
</VirtualHost>
```
Activar virtualHost creado:
```bash
sudo a2ensite api.iagram.mobileia.com.conf
```
Reiniciar Apache:
```bash
sudo service apache2 restart
```



## Creación de Modulo/Libreria para implementar con Composer
1. Crear repositorio en github
2. Clonar repositorio vacio.
3. Crear archivo composer.json:
```json
{
    "name": "mobileia/mia-eloquent-expressive",
    "description": "The library for Zend Expressive 3",
    "version": "0.0.1",
    "minimum-stability": "stable",
    "license": "MIT",
    "authors": [{
        "name": "Matias Camiletti",
        "homepage": "http://www.mobileia.com"
    }],
    "repositories": [{
        "type": "git",
        "url": "https://github.com/MobileIA/mia-eloquent-expressive.git"
    }],
    "require": {
        "php": "^7.1",
        "psr/container": "^1.0",
        "illuminate/database": "^5.6"
    },
    "autoload": {
        "psr-4": {
            "Mobileia\\Expressive\\Database": "src/"
        }
    }
}
```
4. Comitiar y pushear cambios.
5. Crear release.

## Como utilizar Cronjob en Linux
1. Abrir archivo:
```bash
sudo crontab -e
```
2. Agregar linea de Cron:
```
# Se va a ejecutar todos los dias a las 10:00
0 10 * * * wget http://api.iagram.mobileia.com/instagram/test-1 >/dev/null 2>&1
# Se ejecuta cada 5 minutos
*/5 * * * * wget http://api.iagram.mobileia.com/instagram/test-2 >/dev/null 2>&1 

```
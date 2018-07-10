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
3. Configuración de la base de datos:
```bash
composer require illuminate/database
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
use Tuupola\Middleware\CorsMiddleware;

//$app->pipe(CorsMiddleware::class);
$app->pipe(new Tuupola\Middleware\CorsMiddleware([
    "origin" => ["*"],
    "methods" => ["GET", "POST", "PUT", "PATCH", "DELETE"],
    "headers.allow" => ["Authorization", "If-Match", "If-Unmodified-Since", "Content-Type", "Accept", "Origin", "X-Auth-Token", "X-Requested-With", "Access-Control-Allow-Headers", "Access-Control-Allow-Origin"],
    "headers.expose" => ["Etag"],
    "credentials" => false,
    "cache" => 0,
]));
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
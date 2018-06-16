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
1. Limpiar cache
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
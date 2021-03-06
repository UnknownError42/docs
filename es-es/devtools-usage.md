---
layout: article
language: 'es-es'
version: '4.0'
---
##### This article reflects v3.4 and has not yet been revised

{:.alert .alert-danger}

<a name='overview'></a>

# Herramientas para desarrolladores de Phalcon

These tools are a collection of useful scripts to generate skeleton code. Core components of your application can be generated with a simple command, allowing you to easily develop applications using Phalcon.

<h5 class='alert alert-danger'>If you prefer to use the web version instead of the console, this <a href="https://blog.phalconphp.com/post/dont-like-command-line-and-consoles-no-problem">blog post</a> offers more information. </h5>

<a name='download'></a>

## Descargar

You can download or clone a cross platform package containing the developer tools from [Github](https://github.com/phalcon/phalcon-devtools).

<a name='installation'></a>

## Instalación

These are detailed instructions on how to install the developer tools on different platforms:

[Linux](/[/4.0/en/devtools-installation#installation-linux) : [MacOS](/4.0/en/devtools-installation#installation-macos) : [Windows](/4.0/en/devtools-installation#installation-windows)

<a name='available-commands'></a>

## Comandos disponibles

You can get a list of available commands in Phalcon tools by typing: :code:`phalcon commands`

```bash
$ phalcon commands

Phalcon DevTools (3.0.0)

Comandos disponibles:
  commands         (alias de: list, enumerate)
  controller       (alias de: create-controller)
  module           (alias de: create-module)
  model            (alias de: create-model)
  all-models       (alias de: create-all-models)
  project          (alias de: create-project)
  scaffold         (alias de: create-scaffold)
  migration        (alias de: create-migration)
  webtools         (alias de: create-webtools)
```

<a name='project-skeleton'></a>

## Generar un esqueleto de proyecto

You can use Phalcon tools to generate pre-defined project skeletons for your applications with Phalcon framework. By default the project skeleton generator will use mod_rewrite for Apache. Type the following command on your web server document root:

```bash
$ pwd

/Applications/MAMP/htdocs

$ phalcon create-project store
```

The above recommended project structure was generated:

![](/assets/images/content/devtools-usage-01.png)

You could add the parameter `--help` to get help on the usage of a certain script:

```bash
$ phalcon project --help

Phalcon DevTools (3.0.0)

Ayuda:
  Crear un proyecto

Uso:
  project [name] [type] [directory] [enable-webtools]

Argumentos:
  help    Muestra este texto de ayuda

Ejemplo
  phalcon project store simple

Opciones:
 --name               Nombre del nuevo proyecto
 --enable-webtools    Determina si las webtools deben estar activas [opcional]
 --directory=s        Directorio donde el proyecto debe ser creado [opcional]
 --type=s             El tipo de aplicación a generar, opciones: cli, micro, simple, modules
 --template-path=s    Especificar la ruta del template [opcional]
 --use-config-ini     Usar un archivo ini como archivo de configuración [opcional]
 --trace              Mostrar las trazas del framework en caso de alguna excepción. [opcional]
 --help               Muestra esta ayuda[optional]
```

Accessing the project from the web server will show you:

![](/assets/images/content/devtools-usage-02.png)

<a name='generating-controllers'></a>

## Generando controladores

The command `create-controller` generates controller skeleton structures. It's important to invoke this command inside a directory that already has a Phalcon project.

```bash
$ phalcon create-controller --name prueba
```

The following code is generated by the script:

```php
<?php

use Phalcon\Mvc\Controller;

class PruebaController extends Controller
{
    public function indexAction()
    {

    }
}
```

<a name='database-settings'></a>

## Preparando la configuración de la base de datos

When a project is generated using developer tools. A configuration file can be found in `app/config/config.php`. To generate models or scaffold, you will need to change the settings used to connect to your database.

Change the database section in your config.php file:

```php
<?php
defined('BASE_PATH') || define('BASE_PATH', getenv('BASE_PATH') ?: realpath(dirname(__FILE__) . '/../..'));
defined('APP_PATH') || define('APP_PATH', BASE_PATH . '/app');

return new \Phalcon\Config([
    'database' => [
        'adapter'     => 'Mysql',
        'host'        => 'localhost',
        'username'    => 'root',
        'password'    => 'secret',
        'dbname'      => 'test',
        'charset'     => 'utf8',
    ],
    'application' => [
        'appDir'         => APP_PATH . '/',
        'controllersDir' => APP_PATH . '/controllers/',
        'modelsDir'      => APP_PATH . '/models/',
        'migrationsDir'  => APP_PATH . '/migrations/',
        'viewsDir'       => APP_PATH . '/views/',
        'pluginsDir'     => APP_PATH . '/plugins/',
        'libraryDir'     => APP_PATH . '/library/',
        'cacheDir'       => BASE_PATH . '/cache/',

        // Esto permite que la baseUri entienda las rutas del proyecto que no están en el 
        // directorio raíz del espacio web.  Esto se romperá si se mueve el punto de entrada public/index.php o 
        // posiblemente si se cambian las reglas de reescritura del servidor web. Esto también se puede establecer en una ruta estática.
        'baseUri'        => preg_replace('/public([\/\\])index.php$/', '', $_SERVER["PHP_SELF"]),
    ]
]);
```

<a name='generating-models'></a>

## Generando modelos

There are several ways to create models. You can create all models from the default database connection or some selectively. Models can have public attributes for the field representations or setters/getters can be used.

```bash
Opciones:
 --name=s             Nombre de la tabla
 --schema=s           Nombre del esquema. [opcional]
 --namespace=s        Espacio de nombres de los modelos [opcional]
 --get-set            Los atributos deben ser protegidos y tener setters y getters. [opcional]
 --extends=s          Los modelos extienden del nombre de clase dado [opcional]
 --excludefields=l    Excluir campos definidos en la lista separada por comas [opcional]
 --doc                Ayuda a la mejorar el completado de código en IDEs [opcional]
 --directory=s        Directorio base donde se creará el proyecto [opcional]
 --force              Reescribir el modelo. [opcional]
 --trace              Muestra la traza en caso de excepción del framework. [opcional]
 --mapcolumn          Obtener un código para el mapa de columnas. [opcional]
 --abstract           Modelo abstracto [opcional]
```

The simplest way to generate a model is:

```bash
$ phalcon model products
```

```bash
$ phalcon model --name nombreDeLaTabla
```

All table fields are declared public for direct access.

```php
<?php

use Phalcon\Mvc\Model;

class Products extends Model
{
    /**
     * @var integer
     */
    public $id;

    /**
     * @var integer
     */
    public $typesId;

    /**
     * @var string
     */
    public $name;

    /**
     * @var string
     */
    public $price;

    /**
     * @var integer
     */
    public $quantity;

    /**
     * @var string
     */
    public $status;
}
```

By adding the `--get-set` you can generate the fields with protected variables and public setter/getter methods. Those methods can help in business logic implementation within the setter/getter methods.

```php
<?php

use Phalcon\Mvc\Model;

class Products extends Model
{
    /**
     * @var integer
     */
    protected $id;

    /**
     * @var integer
     */
    protected $typesId;

    /**
     * @var string
     */
    protected $name;

    /**
     * @var string
     */
    protected $price;

    /**
     * @var integer
     */
    protected $quantity;

    /**
     * @var string
     */
    protected $status;


    /**
     * Este método establece el valor del campo id
     *
     * @param integer $id
     */
    public function setId($id)
    {
        $this->id = $id;
    }

    /**
     * Este método establece el valor del campo typesId
     *
     * @param integer $typesId
     */
    public function setTypesId($typesId)
    {
        $this->typesId = $typesId;
    }

    // ...

    /**
     * Returna el valor del campo status
     *
     * @return string
     */
    public function getStatus()
    {
        return $this->status;
    }
}
```

A nice feature of the model generator is that it keeps changes made by the developer between code generations. This allows the addition or removal of fields and properties, without worrying about losing changes made to the model itself. The following screencast shows you how it works:

<div align="center">
    <iframe src="https://player.vimeo.com/video/39213020" width="500" height="266" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
</div>

<a name='crud'></a>

## Andamiaje CRUD

Scaffolding is a quick way to generate some of the major pieces of an application. If you want to create the models, views, and controllers for a new resource in a single operation, scaffolding is the tool for the job.

Once the code is generated, it will have to be customized to meet your needs. Many developers avoid scaffolding entirely, opting to write all or most of their source code from scratch. The generated code can serve as a guide to better understand of how the framework works or develop prototypes. The code below shows a scaffold based on the table `products`:

```bash
$ phalcon scaffold --table-name products
```

The scaffold generator will build several files in your application, along with some folders. Here's a quick overview of what will be generated:

| Archivo                                  | Propósito                                 |
| ---------------------------------------- | ----------------------------------------- |
| `app/controllers/ProductsController.php` | El controlador de productos               |
| `app/models/Products.php`                | El modelo de productos                    |
| `app/views/layout/products.phtml`        | La plantilla del controlador de productos |
| `app/views/products/new.phtml`           | La vista de la acción `new`               |
| `app/views/products/edit.phtml`          | La vista de la acción `edit`              |
| `app/views/products/search.phtml`        | La vista de la acción `search`            |

When browsing the recently generated controller, you will see a search form and a link to create a new Product:

![](/assets/images/content/devtools-usage-03.png)

The `create page` allows you to create products applying validations on the Products model. Phalcon will automatically validate not null fields producing warnings if any of them is required.

![](/assets/images/content/devtools-usage-04.png)

After performing a search, a pager component is available to show paged results. Use the "Edit" or "Delete" links in front of each result to perform such actions.

![](/assets/images/content/devtools-usage-05.png)

<a name='web-interface'></a>

## Interfaz web para herramientas

Also, if you prefer, it's possible to use Phalcon Developer Tools from a web interface. Check out the following screencast to figure out how it works:

<div align="center">
<iframe src="https://player.vimeo.com/video/42367665" width="500" height="266" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen mark="crwd-mark"></iframe>
</div>

<a name='phpstorm-ide'></a>

## Integrando las herramientas en PhpStorm IDE

The screencast below shows how to integrate developer tools with the [PhpStorm IDE](https://www.jetbrains.com/phpstorm/). The configuration steps could be easily adapted to other IDEs for PHP.

<div align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/UbUx_6Cs6r4" frameborder="0" allowfullscreen mark="crwd-mark"></iframe>
</div>

<a name='conclusion'></a>

## Conclusión

Phalcon Developer Tools provides an easy way to generate code for your application, reducing development time and potential coding errors.
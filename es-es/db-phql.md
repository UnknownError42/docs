---
layout: article
language: 'es-es'
version: '4.0'
---
##### This article reflects v3.4 and has not yet been revised

{:.alert .alert-danger}

<a name='overview'></a>

# Lenguaje de consulta de Phalcon (PHQL)

Phalcon Query Language, PhalconQL or simply PHQL is a high-level, object-oriented SQL dialect that allows to write queries using a standardized SQL-like language. PHQL is implemented as a parser (written in C) that translates syntax in that of the target RDBMS.

To achieve the highest performance possible, Phalcon provides a parser that uses the same technology as [SQLite](https://en.wikipedia.org/wiki/Lemon_Parser_Generator). This technology provides a small in-memory parser with a very low memory footprint that is also thread-safe.

The parser first checks the syntax of the pass PHQL statement, then builds an intermediate representation of the statement and finally it converts it to the respective SQL dialect of the target RDBMS.

In PHQL, we've implemented a set of features to make your access to databases more secure:

* Los parámetros enlazados son parte del lenguaje PHQL que lo ayudara a proteger su código
* PHQL solo permite una sentencia SQL para ser ejecutada por llamada, previniendo de inyecciones
* PHQL ignora todos los comentarios SQL que generalmente se utilizan en las inyecciones SQL
* PHQL sólo permite declaraciones de manipulación de datos, evitando alterar o borrar tablas y bases de datos por error o desde el exterior sin autorización
* PHQL implementa una abstracción de alto nivel que permite manejar tablas como modelos y campos como atributos de clase

<a name='usage'></a>

## Ejemplo de Uso

To better explain how PHQL works consider the following example. We have two models `Cars` and `Brands`:

```php
<?php

use Phalcon\Mvc\Model;

class Cars extends Model
{
    public $id;

    public $name;

    public $brand_id;

    public $price;

    public $year;

    public $style;

    /**
     * Este modelo se completa desde la tabla sample_cars
     */
    public function getSource()
    {
        return 'sample_cars';
    }

    /**
     * Un automóvil tiene una sola marca, pero una marca tiene muchos automóviles
     */
    public function initialize()
    {
        $this->belongsTo('brand_id', 'Brands', 'id');
    }
}
```

And every Car has a Brand, so a Brand has many Cars:

```php
<?php

use Phalcon\Mvc\Model;

class Brands extends Model
{
    public $id;

    public $name;

    /**
     * El modelo Brands se completa desde la tabla 'sample_brands'
     */
    public function getSource()
    {
        return 'sample_brands';
    }

    /**
     * Una marca tiene muchos automóviles
     */
    public function initialize()
    {
        $this->hasMany('id', 'Cars', 'brand_id');
    }
}
```

<a name='creating'></a>

## Crear consultas PHQL

PHQL queries can be created just by instantiating the class [Phalcon\Mvc\Model\Query](api/Phalcon_Mvc_Model_Query):

```php
<?php

use Phalcon\Mvc\Model\Query;

// Instanciar a Query
$query = new Query(
    'SELECT * FROM Cars',
    $this->getDI()
);

// Ejecutar una consulta retornando un resultado si lo hay
$cars = $query->execute();
```

From a controller or a view, it's easy to create/execute them using an injected `models manager` ([Phalcon\Mvc\Model\Manager](api/Phalcon_Mvc_Model_Manager)):

```php
<?php

// Ejecutando una consulta simple
$query = $this->modelsManager->createQuery('SELECT * FROM Cars');
$cars  = $query->execute();

// Con parámetros enlazados
$query = $this->modelsManager->createQuery('SELECT * FROM Cars WHERE name = :name:');
$cars  = $query->execute(
    [
        'name' => 'Audi',
    ]
);
```

Or simply execute it:

```php
<?php

// Ejecutando una simple consulta
$cars = $this->modelsManager->executeQuery(
    'SELECT * FROM Cars'
);

// Ejecutando con parámetros enlazados
$cars = $this->modelsManager->executeQuery(
    'SELECT * FROM Cars WHERE name = :name:',
    [
        'name' => 'Audi',
    ]
);
```

<a name='selecting-records'></a>

## Seleccionando registros

As the familiar SQL, PHQL allows querying of records using the SELECT statement we know, except that instead of specifying tables, we use the models classes:

```php
<?php

$query = $manager->createQuery(
    'SELECT * FROM Cars ORDER BY Cars.name'
);

$query = $manager->createQuery(
    'SELECT Cars.name FROM Cars ORDER BY Cars.name'
);
```

Classes in namespaces are also allowed:

```php
<?php

$phql  = 'SELECT * FROM Formula\Cars ORDER BY Formula\Cars.name';
$query = $manager->createQuery($phql);

$phql  = 'SELECT Formula\Cars.name FROM Formula\Cars ORDER BY Formula\Cars.name';
$query = $manager->createQuery($phql);

$phql  = 'SELECT c.name FROM Formula\Cars c ORDER BY c.name';
$query = $manager->createQuery($phql);
```

Most of the SQL standard is supported by PHQL, even nonstandard directives such as LIMIT:

```php
<?php

$phql = 'SELECT c.name FROM Cars AS c WHERE c.brand_id = 21 ORDER BY c.name LIMIT 100';

$query = $manager->createQuery($phql);
```

<a name='result-types'></a>

### Tipo de resultado

Depending on the type of columns we query, the result type will vary. If you retrieve a single whole object, then the object returned is a [Phalcon\Mvc\Model\Resultset\Simple](api/Phalcon_Mvc_Model_Resultset_Simple). This kind of resultset is a set of complete model objects:

```php
<?php

$phql = 'SELECT c.* FROM Cars AS c ORDER BY c.name';

$cars = $manager->executeQuery($phql);

foreach ($cars as $car) {
    echo 'Nombre: ', $car->name, "\n";
}
```

This is exactly the same as:

```php
<?php

$cars = Cars::find(
    [
        'order' => 'name'
    ]
);

foreach ($cars as $car) {
    echo 'Nombre: ', $car->name, "\n";
}
```

Complete objects can be modified and re-saved in the database because they represent a complete record of the associated table. There are other types of queries that do not return complete objects, for example:

```php
<?php

$phql = 'SELECT c.id, c.name FROM Cars AS c ORDER BY c.name';

$cars = $manager->executeQuery($phql);

foreach ($cars as $car) {
    echo 'Nombre: ', $car->name, "\n";
}
```

We are only requesting some fields in the table, therefore those cannot be considered an entire object, so the returned object is still a resultset of type [Phalcon\Mvc\Model\Resultset\Simple](api/Phalcon_Mvc_Model_Resultset_Simple). However, each element is a standard object that only contain the two columns that were requested.

These values that don't represent complete objects are what we call scalars. PHQL allows you to query all types of scalars: fields, functions, literals, expressions, etc..:

```php
<?php

$phql = "SELECT CONCAT(c.id, ' ', c.name) AS id_name FROM Cars AS c ORDER BY c.name";

$cars = $manager->executeQuery($phql);

foreach ($cars as $car) {
    echo $car->id_name, "\n";
}
```

As we can query complete objects or scalars, we can also query both at once:

```php
<?php

$phql = 'SELECT c.price*0.16 AS taxes, c.* FROM Cars AS c ORDER BY c.name';

$result = $manager->executeQuery($phql);
```

The result in this case is an object [Phalcon\Mvc\Model\Resultset\Complex](api/Phalcon_Mvc_Model_Resultset_Complex). This allows access to both complete objects and scalars at once:

```php
<?php

foreach ($result as $row) {
    echo 'Nombre: ', $row->cars->name, "\n";
    echo 'Precio: ', $row->cars->price, "\n";
    echo 'Impuestos: ', $row->taxes, "\n";
}
```

` Scalars are mapped as properties of each 'row', while complete objects are mapped as properties with the name of its related model.

<a name='joins'></a>

### Uniones (Joins)

It's easy to request records from multiple models using PHQL. Most kinds of Joins are supported. As we defined relationships in the models, PHQL adds these conditions automatically:

```php
<?php

$phql = 'SELECT Cars.name AS car_name, Brands.name AS brand_name FROM Cars JOIN Brands';

$rows = $manager->executeQuery($phql);

foreach ($rows as $row) {
    echo $row->car_name, "\n";
    echo $row->brand_name, "\n";
}
```

By default, an INNER JOIN is assumed. You can specify the type of JOIN in the query:

```php
<?php

$phql = 'SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands';
$rows = $manager->executeQuery($phql);

$phql = 'SELECT Cars.*, Brands.* FROM Cars LEFT JOIN Brands';
$rows = $manager->executeQuery($phql);

$phql = 'SELECT Cars.*, Brands.* FROM Cars LEFT OUTER JOIN Brands';
$rows = $manager->executeQuery($phql);

$phql = 'SELECT Cars.*, Brands.* FROM Cars CROSS JOIN Brands';
$rows = $manager->executeQuery($phql);
```

It is also possible to manually set the conditions of the JOIN:

```php
<?php

$phql = 'SELECT Cars.*, Brands.* FROM Cars INNER JOIN Brands ON Brands.id = Cars.brands_id';

$rows = $manager->executeQuery($phql);
```

Also, the joins can be created using multiple tables in the FROM clause:

```php
<?php

$phql = 'SELECT Cars.*, Brands.* FROM Cars, Brands WHERE Brands.id = Cars.brands_id';

$rows = $manager->executeQuery($phql);

foreach ($rows as $row) {
    echo 'Automóvil: ', $row->cars->name, "\n";
    echo 'Marca: ', $row->brands->name, "\n";
}
```

If an alias is used to rename the models in the query, those will be used to name the attributes in the every row of the result:

```php
<?php

$phql = 'SELECT c.*, b.* FROM Cars c, Brands b WHERE b.id = c.brands_id';

$rows = $manager->executeQuery($phql);

foreach ($rows as $row) {
    echo 'Automóvil: ', $row->c->name, "\n";
    echo 'Marca: ', $row->b->name, "\n";
}
```

When the joined model has a many-to-many relation to the `from` model, the intermediate model is implicitly added to the generated query:

```php
<?php

$phql = 'SELECT Artists.name, Songs.name FROM Artists ' .
        'JOIN Songs WHERE Artists.genre = "Trip-Hop"';

$result = $this->modelsManager->executeQuery($phql);
```

This code executes the following SQL in MySQL:

```sql
SELECT `artists`.`name`, `songs`.`name` FROM `artists`
INNER JOIN `albums` ON `albums`.`artists_id` = `artists`.`id`
INNER JOIN `songs` ON `albums`.`songs_id` = `songs`.`id`
WHERE `artists`.`genre` = 'Trip-Hop'
```

<a name='aggregations'></a>

### Agregaciones

The following examples show how to use aggregations in PHQL:

```php
<?php

// ¿Cuánto cuestan todos los automóviles?
$phql = 'SELECT SUM(price) AS summatory FROM Cars';
$row  = $manager->executeQuery($phql)->getFirst();
echo $row['summatory'];

// ¿Cuántos automóviles tiene cada marca?
$phql = 'SELECT Cars.brand_id, COUNT(*) FROM Cars GROUP BY Cars.brand_id';
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->brand_id, ' ', $row['1'], "\n";
}

// ¿Cuántos automóviles tiene cada marca?
$phql = 'SELECT Brands.name, COUNT(*) FROM Cars JOIN Brands GROUP BY 1';
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->name, ' ', $row['1'], "\n";
}

$phql = 'SELECT MAX(price) AS maximum, MIN(price) AS minimum FROM Cars';
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row['maximum'], ' ', $row['minimum'], "\n";
}

// Contar marcas usadas distintas
$phql = 'SELECT COUNT(DISTINCT brand_id) AS brandId FROM Cars';
$rows = $manager->executeQuery($phql);
foreach ($rows as $row) {
    echo $row->brandId, "\n";
}
```

<a name='conditions'></a>

### Condiciones

Conditions allow us to filter the set of records we want to query. The `WHERE` clause allows to do that:

```php
<?php

// Condiciones simples
$phql = 'SELECT * FROM Cars WHERE Cars.name = "Lamborghini Espada"';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.price > 10000';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE TRIM(Cars.name) = "Audi R8"';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.name LIKE "Ferrari%"';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.name NOT LIKE "Ferrari%"';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.price IS NULL';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.id IN (120, 121, 122)';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.id NOT IN (430, 431)';
$cars = $manager->executeQuery($phql);

$phql = 'SELECT * FROM Cars WHERE Cars.id BETWEEN 1 AND 100';
$cars = $manager->executeQuery($phql);
```

Also, as part of PHQL, prepared parameters automatically escape the input data, introducing more security:

```php
<?php

$phql = 'SELECT * FROM Cars WHERE Cars.name = :name:';
$cars = $manager->executeQuery(
    $phql,
    [
        'name' => 'Lamborghini Espada'
    ]
);

$phql = 'SELECT * FROM Cars WHERE Cars.name = ?0';
$cars = $manager->executeQuery(
    $phql,
    [
        0 => 'Lamborghini Espada'
    ]
);
```

<a name='inserting-data'></a>

## Insertando datos

With PHQL it's possible to insert data using the familiar INSERT statement:

```php
<?php

// Insertando sin columnas
$phql = 'INSERT INTO Cars VALUES (NULL, "Lamborghini Espada", '
      . '7, 10000.00, 1969, "Grand Tourer")';
$manager->executeQuery($phql);

// Especificando las columnas del insertado
$phql = 'INSERT INTO Cars (name, brand_id, year, style) '
      . 'VALUES ("Lamborghini Espada", 7, 1969, "Grand Tourer")';
$manager->executeQuery($phql);

// Insertando utilizando marcadores
$phql = 'INSERT INTO Cars (name, brand_id, year, style) '
      . 'VALUES (:name:, :brand_id:, :year:, :style)';
$manager->executeQuery(
    $phql,
    [
        'name'     => 'Lamborghini Espada',
        'brand_id' => 7,
        'year'     => 1969,
        'style'    => 'Grand Tourer',
    ]
);
```

Phalcon doesn't only transform the PHQL statements into SQL. All events and business rules defined in the model are executed as if we created individual objects manually. Let's add a business rule on the model cars. A car cannot cost less than $ 10,000:

```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Message;

class Cars extends Model
{
    public function beforeCreate()
    {
        if ($this->price < 10000) {
            $this->appendMessage(
                new Message('Un automóvil no puede costar menos de $ 10,000')
            );

            return false;
        }
    }
}
```

If we made the following `INSERT` in the models Cars, the operation will not be successful because the price does not meet the business rule that we implemented. By checking the status of the insertion we can print any validation messages generated internally:

```php
<?php

$phql = "INSERT INTO Cars VALUES (NULL, 'Nissan Versa', 7, 9999.00, 2015, 'Sedan')";

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    foreach ($result->getMessages() as $message) {
        echo $message->getMessage();
    }
}
```

<a name='updating-data'></a>

## Actualizando datos

Updating rows is very similar than inserting rows. As you may know, the instruction to update records is UPDATE. When a record is updated the events related to the update operation will be executed for each row.

```php
<?php

// Actualizando una simple columna
$phql = 'UPDATE Cars SET price = 15000.00 WHERE id = 101';
$manager->executeQuery($phql);

// Actualizando múltiples columnas
$phql = 'UPDATE Cars SET price = 15000.00, type = "Sedan" WHERE id = 101';
$manager->executeQuery($phql);

// Actualizando múltiples filas
$phql = 'UPDATE Cars SET price = 7000.00, type = "Sedan" WHERE brands_id > 5';
$manager->executeQuery($phql);

// Usando marcadores
$phql = 'UPDATE Cars SET price = ?0, type = ?1 WHERE brands_id > ?2';
$manager->executeQuery(
    $phql,
    [
        0 => 7000.00,
        1 => 'Sedan',
        2 => 5,
    ]
);
```

An `UPDATE` statement performs the update in two phases:

* En primer lugar, si el `UPDATE` tiene una cláusula `WHERE` se recuperan todos los objetos que coincidan con estos criterios,
* En segundo lugar, en función de los objetos consultados, actualiza/cambia los atributos solicitados y los almacena en la base de datos relacional

This way of operation allows that events, virtual foreign keys and validations take part of the updating process. In summary, the following code:

```php
<?php

$phql = 'UPDATE Cars SET price = 15000.00 WHERE id > 101';

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    $messages = $result->getMessages();

    foreach ($messages as $message) {
        echo $message->getMessage();
    }
}
```

is somewhat equivalent to:

```php
<?php

$messages = null;

$process = function () use (&$messages) {
    $cars = Cars::find('id > 101');

    foreach ($cars as $car) {
        $car->price = 15000;

        if ($car->save() === false) {
            $messages = $car->getMessages();

            return false;
        }
    }

    return true;
};

$success = $process();
```

<a name='deleting-data'></a>

## Borrando datos

When a record is deleted the events related to the delete operation will be executed for each row:

```php
<?php

// Borrando una fila simple
$phql = 'DELETE FROM Cars WHERE id = 101';
$manager->executeQuery($phql);

// Borrando múltiples filas
$phql = 'DELETE FROM Cars WHERE id > 100';
$manager->executeQuery($phql);

// Usando marcadores
$phql = 'DELETE FROM Cars WHERE id BETWEEN :initial: AND :final:';
$manager->executeQuery(
    $phql,
    [
        'initial' => 1,
        'final'   => 100,
    ]
);
```

`DELETE` operations are also executed in two phases like `UPDATEs`. To check if the deletion produces any validation messages you should check the status code returned:

```php
<?php

// Borrando múltiples filas
$phql = 'DELETE FROM Cars WHERE id > 100';

$result = $manager->executeQuery($phql);

if ($result->success() === false) {
    $messages = $result->getMessages();

    foreach ($messages as $message) {
        echo $message->getMessage();
    }
}
```

<a name='query-builder'></a>

## Crear consultas utilizando el generador de consultas

A builder is available to create PHQL queries without the need to write PHQL statements, also providing IDE facilities:

```php
<?php

// Obteniendo un conjunto completo
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->join('RobotsParts')
    ->orderBy('Robots.name')
    ->getQuery()
    ->execute();

// Obteniendo la primer fila
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->join('RobotsParts')
    ->orderBy('Robots.name')
    ->getQuery()
    ->getSingleResult();
```

That is the same as:

```php
<?php

$phql = 'SELECT Robots.* FROM Robots JOIN RobotsParts p ORDER BY Robots.name LIMIT 20';

$result = $manager->executeQuery($phql);
```

More examples of the builder:

```php
<?php

// 'SELECT Robots.* FROM Robots';
$builder->from('Robots');

// 'SELECT Robots.*, RobotsParts.* FROM Robots, RobotsParts';
$builder->from(
    [
        'Robots',
        'RobotsParts',
    ]
);

// 'SELECT * FROM Robots';
$phql = $builder->columns('*')
                ->from('Robots');

// 'SELECT id FROM Robots';
$builder->columns('id')
        ->from('Robots');

// 'SELECT id, name FROM Robots';
$builder->columns(['id', 'name'])
        ->from('Robots');

// 'SELECT Robots.* FROM Robots WHERE Robots.name = 'Voltron'';
$builder->from('Robots')
        ->where("Robots.name = 'Voltron'");

// 'SELECT Robots.* FROM Robots WHERE Robots.id = 100';
$builder->from('Robots')
        ->where(100);

// 'SELECT Robots.* FROM Robots WHERE Robots.type = 'virtual' AND Robots.id > 50';
$builder->from('Robots')
        ->where("type = 'virtual'")
        ->andWhere('id > 50');

// 'SELECT Robots.* FROM Robots WHERE Robots.type = 'virtual' OR Robots.id > 50';
$builder->from('Robots')
        ->where("type = 'virtual'")
        ->orWhere('id > 50');

// 'SELECT Robots.* FROM Robots GROUP BY Robots.name';
$builder->from('Robots')
        ->groupBy('Robots.name');

// 'SELECT Robots.* FROM Robots GROUP BY Robots.name, Robots.id';
$builder->from('Robots')
        ->groupBy(['Robots.name', 'Robots.id']);

// 'SELECT Robots.name, SUM(Robots.price) FROM Robots GROUP BY Robots.name';
$builder->columns(['Robots.name', 'SUM(Robots.price)'])
    ->from('Robots')
    ->groupBy('Robots.name');

// 'SELECT Robots.name, SUM(Robots.price) FROM Robots GROUP BY Robots.name HAVING SUM(Robots.price) > 1000';
$builder->columns(['Robots.name', 'SUM(Robots.price)'])
    ->from('Robots')
    ->groupBy('Robots.name')
    ->having('SUM(Robots.price) > 1000');

// 'SELECT Robots.* FROM Robots JOIN RobotsParts';
$builder->from('Robots')
    ->join('RobotsParts');

// 'SELECT Robots.* FROM Robots JOIN RobotsParts AS p';
$builder->from('Robots')
    ->join('RobotsParts', null, 'p');

// 'SELECT Robots.* FROM Robots JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p';
$builder->from('Robots')
    ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p');

// 'SELECT Robots.* FROM Robots
// JOIN RobotsParts ON Robots.id = RobotsParts.robots_id AS p
// JOIN Parts ON Parts.id = RobotsParts.parts_id AS t';
$builder->from('Robots')
    ->join('RobotsParts', 'Robots.id = RobotsParts.robots_id', 'p')
    ->join('Parts', 'Parts.id = RobotsParts.parts_id', 't');

// 'SELECT r.* FROM Robots AS r';
$builder->addFrom('Robots', 'r');

// 'SELECT Robots.*, p.* FROM Robots, Parts AS p';
$builder->from('Robots')
    ->addFrom('Parts', 'p');

// 'SELECT r.*, p.* FROM Robots AS r, Parts AS p';
$builder->from(['r' => 'Robots'])
        ->addFrom('Parts', 'p');

// 'SELECT r.*, p.* FROM Robots AS r, Parts AS p';
$builder->from(['r' => 'Robots', 'p' => 'Parts']);

// 'SELECT Robots.* FROM Robots LIMIT 10';
$builder->from('Robots')
    ->limit(10);

// 'SELECT Robots.* FROM Robots LIMIT 10 OFFSET 5';
$builder->from('Robots')
        ->limit(10, 5);

// 'SELECT Robots.* FROM Robots WHERE id BETWEEN 1 AND 100';
$builder->from('Robots')
        ->betweenWhere('id', 1, 100);

// 'SELECT Robots.* FROM Robots WHERE id IN (1, 2, 3)';
$builder->from('Robots')
        ->inWhere('id', [1, 2, 3]);

// 'SELECT Robots.* FROM Robots WHERE id NOT IN (1, 2, 3)';
$builder->from('Robots')
        ->notInWhere('id', [1, 2, 3]);

// 'SELECT Robots.* FROM Robots WHERE name LIKE '%Art%';
$builder->from('Robots')
        ->where('name LIKE :name:', ['name' => '%' . $name . '%']);

// 'SELECT r.* FROM Store\Robots WHERE r.name LIKE '%Art%';
$builder->from(['r' => 'Store\Robots'])
        ->where('r.name LIKE :name:', ['name' => '%' . $name . '%']);
```

<a name='query-builder-parameters'></a>

### Enlazando parámetros

Bound parameters in the query builder can be set as the query is constructed or past all at once when executing:

```php
<?php

// Pasando parámetros en la construcción de la consulta
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->where('name = :name:', ['name' => $name])
    ->andWhere('type = :type:', ['type' => $type])
    ->getQuery()
    ->execute();

// Pasando parámetros en la ejecución de la consulta
$robots = $this->modelsManager->createBuilder()
    ->from('Robots')
    ->where('name = :name:')
    ->andWhere('type = :type:')
    ->getQuery()
    ->execute(['name' => $name, 'type' => $type]);
```

<a name='disallow-literals'></a>

## No permitir literales en PHQL

Literals can be disabled in PHQL, this means that directly using strings, numbers and boolean values in PHQL strings will be disallowed. If PHQL statements are created embedding external data on them, this could open the application to potential SQL injections:

```php
<?php

$login  = 'voltron';
$phql   = "SELECT * FROM Models\Users WHERE login = '$login'";
$result = $manager->executeQuery($phql);
```

If `$login` is changed to `' OR '' = '`, the produced PHQL is:

```sql
SELECT * FROM Models\Users WHERE login = '' OR '' = ''
```

Which is always `true` no matter what the login stored in the database is.

If literals are disallowed strings can be used as part of a PHQL statement, thus an exception will be thrown forcing the developer to use bound parameters. The same query can be written in a secure way like this:

```php
<?php

$type   = 'virtual';
$phql   = 'SELECT Robots.* FROM Robots WHERE Robots.type = :type:';
$result = $manager->executeQuery(
    $phql,
    [
        'type' => $type,
    ]
);
```

You can disallow literals in the following way:

```php
<?php

use Phalcon\Mvc\Model;

Model::setup(
    [
        'phqlLiterals' => false
    ]
);
```

Bound parameters can be used even if literals are allowed or not. Disallowing them is just another security decision a developer could take in web applications.

<a name='escaping-reserved-words'></a>

## Escapando palabras reservadas

PHQL has a few reserved words, if you want to use any of them as attributes or models names, you need to escape those words using the cross-database escaping delimiters `[` and `]`:

```php
<?php

$phql   = 'SELECT * FROM [Update]';
$result = $manager->executeQuery($phql);

$phql   = 'SELECT id, [Like] FROM Posts';
$result = $manager->executeQuery($phql);
```

The delimiters are dynamically translated to valid delimiters depending on the database system where the application is currently running on.

<a name='lifecycle'></a>

## Ciclo de vida de PHQL

Being a high-level language, PHQL gives developers the ability to personalize and customize different aspects in order to suit their needs. The following is the life cycle of each PHQL statement executed:

* El PHQL es analizado y convertido en una Representación Intermedia (IR) que es independiente de la implementada por el sistema de base de datos de SQL
* La IR se convierte en SQL válido según el sistema de base de datos asociado al modelo
* PHQL statements are parsed once and cached in memory. Further executions of the same statement result in a slightly faster execution

<a name='raw-sql'></a>

## Usando SQL crudo

A database system could offer specific SQL extensions that aren't supported by PHQL, in this case, a raw SQL can be appropriate:

```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

class Robots extends Model
{
    public static function findByCreateInterval()
    {
        // Una declaración de SQL cruda
        $sql = 'SELECT * FROM robots WHERE id > 0';

        // Modelo base
        $robot = new Robots();

        // Ejecutar la consulta
        return new Resultset(
            null,
            $robot,
            $robot->getReadConnection()->query($sql)
        );
    }
}
```

If raw SQL queries are common in your application a generic method could be added to your model:

```php
<?php

use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Resultset\Simple as Resultset;

class Robots extends Model
{
    public static function findByRawSql($conditions, $params = null)
    {
        // Declaración SQL cruda
        $sql = 'SELECT * FROM robots WHERE $conditions';

        // Modelo base
        $robot = new Robots();

        // Ejecutar la consulta
        return new Resultset(
            null,
            $robot,
            $robot->getReadConnection()->query($sql, $params)
        );
    }
}
```

The above `findByRawSql` could be used as follows:

```php
<?php

$robots = Robots::findByRawSql(
    'id > ?',
    [
        10
    ]
);
```

<a name='troubleshooting'></a>

## Resolución de problemas

Some things to keep in mind when using PHQL:

* Las clases son sensibles a mayúsculas y minúsculas, si una clase no se define con el mismo nombre que fue creada, podría conducir a un comportamiento inesperado en sistemas operativos con los sistemas de archivos que distinguen mayúsculas y minúsculas como Linux.
* Un conjunto de caracteres correcto debe definirse en la conexión para enlazar parámetros con éxito.
* Las clases con alias no se reemplazan por clases con espacios de nombres completos, ya que esto solo ocurre en el código PHP y no dentro de las cadenas.
* Si el renombrado de columna está habilitado, evite el uso de alias de columna con el mismo nombre que las columnas a cambiar de nombre, esto puede confundir la resolución de la consulta.
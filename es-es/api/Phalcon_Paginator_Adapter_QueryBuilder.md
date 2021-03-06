---
layout: article
language: 'es-es'
version: '4.0'
title: 'Phalcon\Paginator\Adapter\QueryBuilder'
---
# Class **Phalcon\Paginator\Adapter\QueryBuilder**

*extends* abstract class [Phalcon\Paginator\Adapter](Phalcon_Paginator_Adapter)

*implements* [Phalcon\Paginator\AdapterInterface](Phalcon_Paginator_AdapterInterface)

[Código fuente en GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/paginator/adapter/querybuilder.zep)

Pagination using a PHQL query builder as source of data

```php
<?php

use Phalcon\Paginator\Adapter\QueryBuilder;

$builder = $this->modelsManager->createBuilder()
                ->columns("id, name")
                ->from("Robots")
                ->orderBy("name");

$paginator = new QueryBuilder(
    [
        "builder" => $builder,
        "limit"   => 20,
        "page"    => 1,
    ]
);

```

## Métodos

public **__construct** (*array* $config)

public **getCurrentPage** ()

Get the current page number

public **setQueryBuilder** ([Phalcon\Mvc\Model\Query\Builder](/4.0/en/api/Phalcon_Mvc_Model_Query_Builder) $builder)

Set query builder object

public **getQueryBuilder** ()

Get query builder object

public **getPaginate** ()

Returns a slice of the resultset to show in the pagination

public **setCurrentPage** (*mixed* $page) inherited from [Phalcon\Paginator\Adapter](/4.0/en/api/Phalcon_Paginator_Adapter)

Establecer el número de página actual

public **setLimit** (*mixed* $limitRows) inherited from [Phalcon\Paginator\Adapter](/4.0/en/api/Phalcon_Paginator_Adapter)

Establecer límite de filas

public **getLimit** () inherited from [Phalcon\Paginator\Adapter](/4.0/en/api/Phalcon_Paginator_Adapter)

Obtener el límite actual de filas
---
layout: article
language: 'es-es'
version: '4.0'
title: 'Phalcon\Assets\Collection'
---
# Class **Phalcon\Assets\Collection**

*implements* [Countable](https://php.net/manual/en/class.countable.php), [Iterator](https://php.net/manual/en/class.iterator.php), [Traversable](https://php.net/manual/en/class.traversable.php)

[Source on Github](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/assets/collection.zep)

Representa una colección de recursos

## Métodos

public **getPrefix** ()

...

public **getLocal** ()

...

public **getResources** ()

...

public **getCodes** ()

...

public **getPosition** ()

...

public **getFilters** ()

...

public **getAttributes** ()

...

public **getJoin** ()

...

public **getTargetUri** ()

...

public **getTargetPath** ()

...

public **getTargetLocal** ()

...

public **getSourcePath** ()

...

public **__construct** ()

Phalcon\Assets\Collection constructor

public **add** ([Phalcon\Assets\Resource](Phalcon_Assets_Resource) $resource)

Agrega un recurso a la colección

public **addInline** ([Phalcon\Assets\Inline](Phalcon_Assets_Inline) $code)

Añade un código en línea a la colección

public **has** ([Phalcon\Assets\ResourceInterface](Phalcon_Assets_ResourceInterface) $resource)

Esto comprueba que el recurso se haya agregado a la colección.

```php
<?php

use Phalcon\Assets\Resource;
use Phalcon\Assets\Collection;

$collection = new Collection();

$resource = new Resource("js", "js/jquery.js");
$resource->has($resource); // true

```

public **addCss** (*mixed* $path, [*mixed* $local], [*mixed* $filter], [*mixed* $attributes])

Adds a CSS resource to the collection

public **addInlineCss** (*mixed* $content, [*mixed* $filter], [*mixed* $attributes])

Adds an inline CSS to the collection

public [Phalcon\Assets\Collection](Phalcon_Assets_Collection) **addJs** (*string* $path, [*boolean* $local], [*boolean* $filter], [*array* $attributes])

Adds a javascript resource to the collection

public **addInlineJs** (*mixed* $content, [*mixed* $filter], [*mixed* $attributes])

Adds an inline javascript to the collection

public **count** ()

Returns the number of elements in the form

public **rewind** ()

Rebobina el iterador interno

public **current** ()

Returns the current resource in the iterator

public *int* **key** ()

Devuelve la llave/posición actual del iterador

public **next** ()

Mueve el puntero interno de iteración a la siguiente posición

public **valid** ()

Check if the current element in the iterator is valid

public **setTargetPath** (*mixed* $targetPath)

Sets the target path of the file for the filtered/join output

public **setSourcePath** (*mixed* $sourcePath)

Sets a base source path for all the resources in this collection

public **setTargetUri** (*mixed* $targetUri)

Sets a target uri for the generated HTML

public **setPrefix** (*mixed* $prefix)

Sets a common prefix for all the resources

public **setLocal** (*mixed* $local)

Sets if the collection uses local resources by default

public **setAttributes** (*array* $attributes)

Sets extra HTML attributes

public **setFilters** (*array* $filters)

Sets an array of filters in the collection

public **setTargetLocal** (*mixed* $targetLocal)

Sets the target local

public **join** (*mixed* $join)

Sets if all filtered resources in the collection must be joined in a single result file

public **getRealTargetPath** (*mixed* $basePath)

Returns the complete location where the joined/filtered collection must be written

public **addFilter** ([Phalcon\Assets\FilterInterface](Phalcon_Assets_FilterInterface) $filter)

Adds a filter to the collection

final protected **addResource** ([Phalcon\Assets\ResourceInterface](Phalcon_Assets_ResourceInterface) $resource)

Adds a resource or inline-code to the collection
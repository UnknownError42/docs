---
layout: article
language: 'es-es'
version: '4.0'
title: 'Phalcon\Logger\AdapterInterface'
---
# Interface **Phalcon\Logger\AdapterInterface**

[Código fuente en GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/logger/adapterinterface.zep)

## Métodos

abstract public **setFormatter** ([Phalcon\Logger\FormatterInterface](/4.0/en/api/Phalcon_Logger_FormatterInterface) $formatter)

...

abstract public **getFormatter** ()

...

abstract public **setLogLevel** (*mixed* $level)

...

abstract public **getLogLevel** ()

...

abstract public **log** (*mixed* $type, [*mixed* $message], [*array* $context])

...

abstract public **begin** ()

...

abstract public **commit** ()

...

abstract public **rollback** ()

...

abstract public **close** ()

...

abstract public **debug** (*mixed* $message, [*array* $context])

...

abstract public **error** (*mixed* $message, [*array* $context])

...

abstract public **info** (*mixed* $message, [*array* $context])

...

abstract public **notice** (*mixed* $message, [*array* $context])

...

abstract public **warning** (*mixed* $message, [*array* $context])

...

abstract public **alert** (*mixed* $message, [*array* $context])

...

abstract public **emergency** (*mixed* $message, [*array* $context])

...
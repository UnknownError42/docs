* * *

layout: article language: 'es-es' version: '4.0' title: 'Phalcon\Mvc\Model\Validator\Numericality'

* * *

# Class **Phalcon\Mvc\Model\Validator\Numericality**

*extends* abstract class [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

*implements* [Phalcon\Mvc\Model\ValidatorInterface](/4.0/en/api/Phalcon_Mvc_Model_ValidatorInterface)

<a href="https://github.com/phalcon/cphalcon/tree/v4.0.0/phalcon/mvc/model/validator/numericality.zep" class="btn btn-default btn-sm">Código fuente en GitHub</a>

Allows to validate if a field has a valid numeric format

This validator is only for use with Phalcon\Mvc\Collection. If you are using Phalcon\Mvc\Model, please use the validators provided by Phalcon\Validation.

```php
<?php

use Phalcon\Mvc\Model\Validator\Numericality as NumericalityValidator;

class Products extends \Phalcon\Mvc\Collection
{
    public function validation()
    {
        $this->validate(
            new NumericalityValidator(
                [
                    "field" => "price",
                ]
            )
        );

        if ($this->validationHasFailed() === true) {
            return false;
        }
    }
}

```

## Métodos

public **validate** ([Phalcon\Mvc\EntityInterface](/4.0/en/api/Phalcon_Mvc_EntityInterface) $record)

Executes the validator

public **__construct** (*array* $options) inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Phalcon\Mvc\Model\Validator constructor

protected **appendMessage** (*string* $message, [*string* | *array* $field], [*string* $type]) inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Añade un mensaje para el validador

public **getMessages** () inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Devuelve mensajes generados por el validador

public *array* **getOptions** () inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Devuelve todas las opciones dela validador

public **getOption** (*mixed* $option, [*mixed* $defaultValue]) inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Devuelve una opción

public **isSetOption** (*mixed* $option) inherited from [Phalcon\Mvc\Model\Validator](/4.0/en/api/Phalcon_Mvc_Model_Validator)

Comprobar si una opción se ha definido en las opciones de validación
---
layout: article
language: 'ja-jp'
version: '4.0'
---
##### This article reflects v3.4 and has not yet been revised

{:.alert .alert-danger}

<a name='working-with'></a>

# モデルの動作

モデルは、アプリケーションの情報 (データ) と、そのデータを操作するためのルールを表します。 モデルは主に、それに対応するテーブルとの対話のルールを管理するために使用されます。 ほとんどの場合、データベース内の各テーブルは、アプリケーション内の1つのモデルと対応します。 アプリケーションのビジネスロジックの大半は、モデルに集中します。

[Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) is the base for all models in a Phalcon application. It provides database independence, basic CRUD functionality, advanced finding capabilities, and the ability to relate models to one another, among other services. [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) avoids the need of having to use SQL statements because it translates methods dynamically to the respective database engine operations.

<h5 class='alert alert-warning'>Models are intended to work with the database on a high layer of abstraction. If you need to work with databases at a lower level check out the <a href="api/Phalcon_Db">Phalcon\Db</a> component documentation.</h5>

<a name='creating'></a>

## モデルの作成

A model is a class that extends from [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model). Its class name should be in camel case notation:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{

}
```

<h5 class='alert alert-warning'>If you're using PHP 5.4/5.5 it is recommended you declare each column that makes part of the model in order to save memory and reduce the memory allocation. </h5>

By default, the model `Store\Toys\RobotParts` will map to the table `robot_parts`. If you want to manually specify another name for the mapped table, you can use the `setSource()` method:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{
    public function initialize()
    {
        $this->setSource('toys_robot_parts');
    }
}
```

The model `RobotParts` now maps to `toys_robot_parts` table. The `initialize()` method helps with setting up this model with a custom behavior i.e. a different table.

The `initialize()` method is only called once during the request. This method is intended to perform initializations that apply for all instances of the model created within the application. If you want to perform initialization tasks for every instance created you can use the `onConstruct()` method:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class RobotParts extends Model
{
    public function onConstruct()
    {
        // ...
    }
}
```

<a name='properties-setters-getters'></a>

### Public properties vs. Setters/Getters

Models can be implemented public properties, meaning that each property can be read/updated from any part of the code that has instantiated that model class:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $price;
}
```

Another implementation is to use getters and setter functions, which control which properties are publicly available for that model. The benefit of using getters and setters is that the developer can perform transformations and validation checks on the values set for the model, which is impossible when using public properties. Additionally getters and setters allow for future changes without changing the interface of the model class. So if a field name changes, the only change needed will be in the private property of the model referenced in the relevant getter/setter and nowhere else in the code.

```php
<?php

namespace Store\Toys;

use InvalidArgumentException;
use Phalcon\Mvc\Model;

class Robots extends Model
{
    protected $id;

    protected $name;

    protected $price;

    public function getId()
    {
        return $this->id;
    }

    public function setName($name)
    {
        // 名前は短すぎないか？
        if (strlen($name) < 10) {
            throw new InvalidArgumentException(
                'The name is too short'
            );
        }

        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setPrice($price)
    {
        // マイナスの価格は許可されない
        if ($price < 0) {
            throw new InvalidArgumentException(
                "Price can't be negative"
            );
        }

        $this->price = $price;
    }

    public function getPrice()
    {
        // 使う前にdouble型に変換
        return (double) $this->price;
    }
}
```

Public properties provide less complexity in development. However getters/setters can heavily increase the testability, extensibility and maintainability of applications. Developers can decide which strategy is more appropriate for the application they are creating, depending on the needs of the application. The ORM is compatible with both schemes of defining properties.

<h5 class='alert alert-warning'>Underscores in property names can be problematic when using getters and setters. </h5>

If you use underscores in your property names, you must still use camel case in your getter/setter declarations for use with magic methods. (e.g. `$model->getPropertyName` instead of `$model->getProperty_name`, `$model->findByPropertyName` instead of `$model->findByProperty_name`, etc.). As much of the system expects camel case, and underscores are commonly removed, it is recommended to name your properties in the manner shown throughout the documentation. You can use a column map (as described above) to ensure proper mapping of your properties to their database counterparts.

<a name='records-to-objects'></a>

## オブジェクトへのレコード格納について理解する

Every instance of a model represents a row in the table. You can easily access record data by reading object properties. For example, for a table 'robots' with the records:

```sql
mysql> select * from robots;
+----+------------+------------+------+
| id | name       | type       | year |
+----+------------+------------+------+
|  1 | Robotina   | mechanical | 1972 |
|  2 | Astro Boy  | mechanical | 1952 |
|  3 | Terminator | cyborg     | 2029 |
+----+------------+------------+------+
3 rows in set (0.00 sec)
```

You could find a certain record by its primary key and then print its name:

```php
<?php

use Store\Toys\Robots;

// id = 3のレコードを検索
$robot = Robots::findFirst(3);

// 'Terminator' を出力
echo $robot->name;
```

一度レコードがメモリ内に格納されれば、データを更新して、その変更を保存することができます:

```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(3);

$robot->name = 'RoboCop';

$robot->save();
```

As you can see, there is no need to use raw SQL statements. [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) provides high database abstraction for web applications.

<a name='finding-records'></a>

## レコードの検索

[Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) also offers several methods for querying records. The following examples will show you how to query one or more records from a model:

```php
<?php

use Store\Toys\Robots;

// どれくらいロボットがいるか？
$robots = Robots::find();
echo 'There are ', count($robots), "\n";

// 機械式のロボットはどれくらいいるか？
$robots = Robots::find("type = 'mechanical'");
echo 'There are ', count($robots), "\n";

// バーチャルなロボットを取得して名前順に並び替えて出力する
$robots = Robots::find(
    [
        "type = 'virtual'",
        'order' => 'name',
    ]
);
foreach ($robots as $robot) {
    echo $robot->name, "\n";
}

// バーチャルなロボットを名前順に並び替えて最初の100件を取得する
$robots = Robots::find(
    [
        "type = 'virtual'",
        'order' => 'name',
        'limit' => 100,
    ]
);
foreach ($robots as $robot) {
   echo $robot->name, "\n";
}
```

<h5 class='alert alert-warning'>If you want find record by external data (such as user input) or variable data you must use <a href="#binding-parameters">Binding Parameters</a>`.</h5>

`findFirst()` メソッドを使用して、指定した条件に一致する最初のレコードのみを取得することもできます。

```php
<?php

use Store\Toys\Robots;

// robotsテーブルの最初のレコードは？
$robot = Robots::findFirst();
echo 'The robot name is ', $robot->name, "\n";

// robotsテーブルの最初の機械式のロボットは？
$robot = Robots::findFirst("type = 'mechanical'");
echo 'The first mechanical robot name is ', $robot->name, "\n";

// バーチャルなロボットを名前順に並び替えて最初の100件を取得する
$robot = Robots::findFirst(
    [
        "type = 'virtual'",
        'order' => 'name',
    ]
);

echo 'The first virtual robot name is ', $robot->name, "\n";
```

`find()` および `findFirst()` メソッドは、検索条件を指定する連想配列を受け付けます:

```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(
    [
        "type = 'virtual'",
        'order' => 'name DESC',
        'limit' => 30,
    ]
);

$robots = Robots::find(
    [
        'conditions' => 'type = ?1',
        'bind'       => [
            1 => 'virtual',
        ]
    ]
);
```

使用可能なクエリのオプションは次のとおりです:

| Parameter     | Description                                                                                                                                          | 例                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `conditions`  | find時の検索条件。 指定された条件を満たすレコードのみを抽出するために使用されます。 By default [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) assumes the first parameter are the conditions. | `'conditions' => "name LIKE 'steve%'"`                            |
| `columns`     | Return specific columns instead of the full columns in the model. When using this option an incomplete object is returned                            | `'columns' => 'id, name'`                                         |
| `bind`        | bindは、プレースホルダをエスケープ値で置き換えるオプションと一緒に使用されるため、セキュリティが強化されます。                                                                                            | `'bind' => ['status' => 'A', 'type' => 'some-time']`        |
| `bindTypes`   | パラメータをバインドする時にこのパラメータを使用すると、バインドされたパラメータをさらに型キャストして、セキュリティをさらに強化することができます。                                                                           | `'bindTypes' => [Column::BIND_PARAM_STR, Column::BIND_PARAM_INT]` |
| `order`       | Is used to sort the resultset. Use one or more fields separated by commas.                                                                           | `'order' => 'name DESC, status'`                                  |
| `limit`       | クエリの結果を特定の範囲に制限します。                                                                                                                                  | `'limit' => 10`                                                   |
| `offset`      | クエリの結果を一定量だけオフセットします。                                                                                                                                | `'offset' => 5`                                                   |
| `group`       | 複数のレコードにわたってデータを収集し、結果を1つ以上の列でグループ化することができます。                                                                                                        | `'group' => 'name, status'`                                       |
| `for_update`  | With this option, [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) reads the latest available data, setting exclusive locks on each row it reads         | `'for_update' => true`                                            |
| `shared_lock` | With this option, [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) reads the latest available data, setting shared locks on each row it reads            | `'shared_lock' => true`                                           |
| `cache`       | 結果セットをキャッシュし、リレーショナルシステムへの継続的なアクセスを減らします。                                                                                                            | `'cache' => ['lifetime' => 3600, 'key' => 'my-find-key']`   |
| `hydration`   | 結果に返された各レコードを表すためのハイドレーション戦略を設定します。                                                                                                                  | `'hydration' => Resultset::HYDRATE_OBJECTS`                       |

If you prefer, there is also available a way to create queries in an object-oriented way, instead of using an array of parameters:

```php
<?php

use Store\Toys\Robots;

$robots = Robots::query()
    ->where('type = :type:')
    ->andWhere('year < 2000')
    ->bind(['type' => 'mechanical'])
    ->order('name')
    ->execute();
```

The static method `query()` returns a [Phalcon\Mvc\Model\Criteria](api/Phalcon_Mvc_Model_Criteria) object that is friendly with IDE autocompleters.

All the queries are internally handled as [PHQL](/4.0/en/db-phql) queries. PHQL is a high-level, object-oriented and SQL-like language. This language provide you more features to perform queries like joining other models, define groupings, add aggregations etc.

Lastly, there is the `findFirstBy<property-name>()` method. This method expands on the `findFirst()` method mentioned earlier. It allows you to quickly perform a retrieval from a table by using the property name in the method itself and passing it a parameter that contains the data you want to search for in that column. An example is in order, so taking our Robots model mentioned earlier:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $price;
}
```

We have three properties to work with here: `$id`, `$name` and `$price`. So, let's say you want to retrieve the first record in the table with the name 'Terminator'. This could be written like:

```php
<?php

use Store\Toys\Robots;

$name = 'Terminator';

$robot = Robots::findFirstByName($name);

if ($robot) {
    echo 'The first robot with the name ' . $name . ' cost ' . $robot->price . '.';
} else {
    echo 'There were no robots found in our table with the name ' . $name . '.';
}
```

Notice that we used 'Name' in the method call and passed the variable `$name` to it, which contains the name we are looking for in our table. Notice also that when we find a match with our query, all the other properties are available to us as well.

<a name='resultsets'></a>

### モデルの結果セット

While `findFirst()` returns directly an instance of the called class (when there is data to be returned), the `find()` method returns a [Phalcon\Mvc\Model\Resultset\Simple](api/Phalcon_Mvc_Model_Resultset_Simple). This is an object that encapsulates all the functionality a resultset has like traversing, seeking specific records, counting, etc.

These objects are more powerful than standard arrays. One of the greatest features of the [Phalcon\Mvc\Model\Resultset](api/Phalcon_Mvc_Model_Resultset) is that at any time there is only one record in memory. This greatly helps in memory management especially when working with large amounts of data.

```php
<?php

use Store\Toys\Robots;

// 全てのロボットを取得
$robots = Robots::find();

// foreachで横断する
foreach ($robots as $robot) {
    echo $robot->name, "\n";
}

// whileで横断する
$robots->rewind();

while ($robots->valid()) {
    $robot = $robots->current();

    echo $robot->name, "\n";

    $robots->next();
}

// 結果セットをカウント
echo count($robots);

// 結果セットをカウントする他の方法
echo $robots->count();

// 内部カーソルを3番目のロボットに移動する
$robots->seek(2);

$robot = $robots->current();

// 結果セットのあるポジションのロボットにアクセス
$robot = $robots[5];

// 特定の位置にレコードがあるかどうかを確認する
if (isset($robots[3])) {
   $robot = $robots[3];
}

// 結果セットの最初のレコードを取得
$robot = $robots->getFirst();

// 最後のレコードを取得
$robot = $robots->getLast();
```

Phalcon's resultsets emulate scrollable cursors, you can get any row just by accessing its position, or seeking the internal pointer to a specific position. Note that some database systems don't support scrollable cursors, this forces to re-execute the query in order to rewind the cursor to the beginning and obtain the record at the requested position. Similarly, if a resultset is traversed several times, the query must be executed the same number of times.

As storing large query results in memory could consume many resources, resultsets are obtained from the database in chunks of 32 rows - reducing the need to re-execute the request in several cases.

Note that resultsets can be serialized and stored in a cache backend. [Phalcon\Cache](api/Phalcon_Cache) can help with that task. However, serializing data causes [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) to retrieve all the data from the database in an array, thus consuming more memory while this process takes place.

```php
<?php

// partsモデルからすべてのレコードを照会します
$parts = Parts::find();

// ファイルに結果セットを保存
file_put_contents(
    'cache.txt',
    serialize($parts)
);

// ファイルからpartsを取得
$parts = unserialize(
    file_get_contents('cache.txt')
);

// partsを走査
foreach ($parts as $part) {
    echo $part->id;
}
```

<a name='custom-resultsets'></a>

### カスタム結果セット

There are times that the application logic requires additional manipulation of the data as it is retrieved from the database. Previously, we would just extend the model and encapsulate the functionality in a class in the model or a trait, returning back to the caller usually an array of transformed data.

With custom resultsets, you no longer need to do that. The custom resultset will encapsulate the functionality that otherwise would be in the model and can be reused by other models, thus keeping the code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). This way, the `find()` method will no longer return the default [Phalcon\Mvc\Model\Resultset](api/Phalcon_Mvc_Model_Resultset), but instead the custom one. Phalcon allows you to do this by using the `getResultsetClass()` in your model.

First we need to define the resultset class:

```php
<?php

namespace Application\Mvc\Model\Resultset;

use \Phalcon\Mvc\Model\Resultset\Simple;

class Custom extends Simple
{
    public function getSomeData() {
        /** CODE */
    }
}
```

In the model, we set the class in the `getResultsetClass()` as follows:

```php
<?php

namespace Phalcon\Test\Models\Statistics;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function getSource()
    {
        return 'robots';
    }

    public function getResultsetClass()
    {
    return 'Application\Mvc\Model\Resultset\Custom';
    }
}
```

and finally in your code you will have something like this:

```php
<?php

/**
 * robotsの検索
 */
$robots = Robots::find(
    [
        'conditions' => 'date between "2017-01-01" AND "2017-12-31"',
        'order'      => 'date'
    ]
);

/**
 * データをビューに渡す
 */
$this->view->mydata = $robots->getSomeData();
```

<a name='filters'></a>

### 結果セットのフィルタリング

The most efficient way to filter data is setting some search criteria, databases will use indexes set on tables to return data faster. Phalcon additionally allows you to filter the data using PHP using any resource that is not available in the database:

```php
<?php

$customers = Customers::find();

$customers = $customers->filter(
    function ($customer) {
        // 有効な電子メールを持っている顧客のみを返します
        if (filter_var($customer->email, FILTER_VALIDATE_EMAIL)) {
            return $customer;
        }
    }
);
```

<a name='binding-parameters'></a>

### パラメーターのバインド

Bound parameters are also supported in [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model). You are encouraged to use this methodology so as to eliminate the possibility of your code being subject to SQL injection attacks. Both string and integer placeholders are supported. パラメータのバインドは、以下のように簡単に実施できます:

```php
<?php

use Store\Toys\Robots;

// 文字列プレースホルダを使用してパラメータをバインドするrobotsテーブル用クエリ
// プレースホルダと同じキーのパラメータ
$robots = Robots::find(
    [
        'name = :name: AND type = :type:',
        'bind' => [
            'name' => 'Robotina',
            'type' => 'maid',
        ],
    ]
);

// 整数のプレースホルダによるパラメータをバインドしたrobotsテーブル用クエリ
$robots = Robots::find(
    [
        'name = ?1 AND type = ?2',
        'bind' => [
            1 => 'Robotina',
            2 => 'maid',
        ],
    ]
);

// 文字列と整数の両方のプレースホルダによるパラメータをバインドしたrobotsテーブル用クエリ
// プレースホルダと同じキーのパラメータ
$robots = Robots::find(
    [
        'name = :name: AND type = ?1',
        'bind' => [
            'name' => 'Robotina',
            1      => 'maid',
        ],
    ]
);
```

When using numeric placeholders, you will need to define them as integers i.e. `1` or `2`. In this case `'1'` or `'2'` are considered strings and not numbers, so the placeholder could not be successfully replaced.

Strings are automatically escaped using [PDO](https://php.net/manual/en/pdo.prepared-statements.php). This function takes into account the connection charset, so its recommended to define the correct charset in the connection parameters or in the database configuration, as a wrong charset will produce undesired effects when storing or retrieving data.

Additionally you can set the parameter `bindTypes`, this allows defining how the parameters should be bound according to its data type:

```php
<?php

use Phalcon\Db\Column;
use Store\Toys\Robots;

// バインドパラメータ
$parameters = [
    'name' => 'Robotina',
    'year' => 2008,
];

// 型キャスト
$types = [
    'name' => Column::BIND_PARAM_STR,
    'year' => Column::BIND_PARAM_INT,
];

// 文字列プレースホルダを使用してパラメータをバインドするrobotsテーブル用クエリ
$robots = Robots::find(
    [
        'name = :name: AND year = :year:',
        'bind'      => $parameters,
        'bindTypes' => $types,
    ]
);
```

<h5 class='alert alert-warning'>Since the default bind-type is <code>Phalcon\Db\Column::BIND_PARAM_STR</code>, there is no need to specify the 'bindTypes' parameter if all of the columns are of that type.</h5>

If you bind arrays in bound parameters, keep in mind, that keys must be numbered from zero:

```php
<?php

use Store\Toys\Robots;

$array = ['a','b','c']; // $array: [[0] => 'a', [1] => 'b', [2] => 'c']

unset($array[1]); // $array: [[0] => 'a', [2] => 'c']

// キーの番号を振り直さなければならない
$array = array_values($array); // $array: [[0] => 'a', [1] => 'c']

$robots = Robots::find(
    [
        'letter IN ({letter:array})',
        'bind' => [
            'letter' => $array
        ]
    ]
);
```

<h5 class='alert alert-warning'>Bound parameters are available for all query methods such as <code>find()</code> and <code>findFirst()</code> but also the calculation methods like <code>count()</code>, <code>sum()</code>, <code>average()</code> etc. </h5>

If you're using "finders" e.g. `find()`, `findFirst()`, etc., bound parameters are automatically used:

```php
<?php

use Store\Toys\Robots;

// バインドパラメータを使用する明示的なクエリ
$robots = Robots::find(
    [
        'name = ?0',
        'bind' => [
            'Ultron',
        ],
    ]
);

// バインドパラメータを使用した暗黙的なクエリ
$robots = Robots::findByName('Ultron');
```

<a name='preparing-records'></a>

## 取得したレコードの初期化/準備

May be the case that after obtaining a record from the database is necessary to initialise the data before being used by the rest of the application. You can implement the `afterFetch()` method in a model, this event will be executed just after create the instance and assign the data to it:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $status;

    public function beforeSave()
    {
        // 配列を文字列に変換
        $this->status = join(',', $this->status);
    }

    public function afterFetch()
    {
        // 文字列を配列に変換
        $this->status = explode(',', $this->status);
    }

    public function afterSave()
    {
        // 文字列を配列に変換
        $this->status = explode(',', $this->status);
    }
}
```

If you use getters/setters instead of/or together with public properties, you can initialize the field once it is accessed:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $id;

    public $name;

    public $status;

    public function getStatus()
    {
        return explode(',', $this->status);
    }
}
```

<a name='calculations'></a>

## 集計の生成

Calculations (or aggregations) are helpers for commonly used functions of database systems such as `COUNT`, `SUM`, `MAX`, `MIN` or `AVG`. [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) allows to use these functions directly from the exposed methods.

Count examples:

```php
<?php

// 従業員は何名？
$rowcount = Employees::count();

// 従業員に割り当てられる担当はいくつあるか？
$rowcount = Employees::count(
    [
        'distinct' => 'area',
    ]
);

// 検査担当の従業員は何名いるか？
$rowcount = Employees::count(
    'area = 'Testing''
);

// 担当別にグルーピングした結果で従業員をカウントする
$group = Employees::count(
    [
        'group' => 'area',
    ]
);
foreach ($group as $row) {
   echo 'There are ', $row->rowcount, ' in ', $row->area;
}

// 従業員を担当別にグルーピングしてカウントし、件数で結果を並び替える
$group = Employees::count(
    [
        'group' => 'area',
        'order' => 'rowcount',
    ]
);

// バインドされたパラメータを使用してSQLインジェクションを避ける
$group = Employees::count(
    [
        'type > ?0',
        'bind' => [
            $type
        ],
    ]
);
```

Sum examples:

```php
<?php

// 全従業員の給与はいくらか？
$total = Employees::sum(
    [
        'column' => 'salary',
    ]
);

// 全ての営業担当の給与はいくらか？
$total = Employees::sum(
    [
        'column'     => 'salary',
        'conditions' => "area = 'Sales'",
    ]
);

// 各担当ごとの給与のグルーピングを生成する
$group = Employees::sum(
    [
        'column' => 'salary',
        'group'  => 'area',
    ]
);
foreach ($group as $row) {
   echo 'The sum of salaries of the ', $row->area, ' is ', $row->sumatory;
}

// 各担当ごとの給与のグルーピングを生成して
// 給与が高いものから低いものへ並べる
$group = Employees::sum(
    [
        'column' => 'salary',
        'group'  => 'area',
        'order'  => 'sumatory DESC',
    ]
);

// バインドされたパラメータを使用してSQLインジェクションを避ける
$group = Employees::sum(
    [
        'conditions' => 'area > ?0',
        'bind'       => [
            $area
        ],
    ]
);
```

Average examples:

```php
<?php

// 全従業員の平均給与はいくらか？
$average = Employees::average(
    [
        'column' => 'salary',
    ]
);

// 営業担当者の平均給与はいくらか？
$average = Employees::average(
    [
        'column'     => 'salary',
        'conditions' => "area = 'Sales'",
    ]
);

// バインドされたパラメータを使用してSQLインジェクションを避ける
$average = Employees::average(
    [
        'column'     => 'age',
        'conditions' => 'area > ?0',
        'bind'       => [
            $area
        ],
    ]
);
```

Max/Min examples:

```php
<?php

// 全従業員で最年長は？
$age = Employees::maximum(
    [
        'column' => 'age',
    ]
);

// 全ての営業担当で最年長は？
$age = Employees::maximum(
    [
        'column'     => 'age',
        'conditions' => "area = 'Sales'",
    ]
);

// 全従業員中の最低給与は？
$salary = Employees::minimum(
    [
        'column' => 'salary',
    ]
);
```

<a name='create-update-records'></a>

## レコードの登録と更新

The `Phalcon\Mvc\Model::save()` method allows you to create/update records according to whether they already exist in the table associated with a model. The save method is called internally by the create and update methods of [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model). For this to work as expected it is necessary to have properly defined a primary key in the entity to determine whether a record should be updated or created.

Also the method executes associated validators, virtual foreign keys and events that are defined in the model:

```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->type = 'mechanical';
$robot->name = 'Astro Boy';
$robot->year = 1952;

if ($robot->save() === false) {
    echo "Umh, We can't store robots right now: \n";

    $messages = $robot->getMessages();

    foreach ($messages as $message) {
        echo $message, "\n";
    }
} else {
    echo 'Great, a new robot was saved successfully!';
}
```

An array could be passed to `save` to avoid assign every column manually. [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) will check if there are setters implemented for the columns passed in the array giving priority to them instead of assign directly the values of the attributes:

```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->save(
    [
        'type' => 'mechanical',
        'name' => 'Astro Boy',
        'year' => 1952,
    ]
);
```

Values assigned directly or via the array of attributes are escaped/sanitized according to the related attribute data type. So you can pass an insecure array without worrying about possible SQL injections:

```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->save($_POST);
```

<h5 class='alert alert-warning'>Without precautions mass assignment could allow attackers to set any database column's value. Only use this feature if you want to permit a user to insert/update every column in the model, even if those fields are not in the submitted form. </h5>

You can set an additional parameter in `save` to set a whitelist of fields that only must taken into account when doing the mass assignment:

```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->save(
    $_POST,
    [
        'name',
        'type',
    ]
);
```

<a name='create-update-with-confidence'></a>

### 確実に作成/更新する

When an application has a lot of competition, we could be expecting create a record but it is actually updated. This could happen if we use `Phalcon\Mvc\Model::save()` to persist the records in the database. If we want to be absolutely sure that a record is created or updated, we can change the `save()` call with `create()` or `update()`:

```php
<?php

use Store\Toys\Robots;

$robot = new Robots();

$robot->type = 'mechanical';
$robot->name = 'Astro Boy';
$robot->year = 1952;

// このレコードは作成されなければいけない
if ($robot->create() === false) {
    echo "Umh, We can't store robots right now: \n";

    $messages = $robot->getMessages();

    foreach ($messages as $message) {
        echo $message, "\n";
    }
} else {
    echo 'Great, a new robot was created successfully!';
}
```

The methods `create` and `update` also accept an array of values as parameter.

<a name='delete-records'></a>

## レコードの削除

The `Phalcon\Mvc\Model::delete()` method allows to delete a record. You can use it as follows:

```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst(11);

if ($robot !== false) {
    if ($robot->delete() === false) {
        echo "Sorry, we can't delete the robot right now: \n";

        $messages = $robot->getMessages();

        foreach ($messages as $message) {
            echo $message, "\n";
        }
    } else {
        echo 'The robot was deleted successfully!';
    }
}
```

You can also delete many records by traversing a resultset with a `foreach`:

```php
<?php

use Store\Toys\Robots;

$robots = Robots::find(
    "type = 'mechanical'"
);

foreach ($robots as $robot) {
    if ($robot->delete() === false) {
        echo "Sorry, we can't delete the robot right now: \n";

        $messages = $robot->getMessages();

        foreach ($messages as $message) {
            echo $message, "\n";
        }
    } else {
        echo 'The robot was deleted successfully!';
    }
}
```

削除が処理される際に、カスタムビジネスルールの実行を定義するには、次のイベントを使用します。

| Operation | Name         | Can stop operation? | Explanation                              |
| --------- | ------------ |:-------------------:| ---------------------------------------- |
| 削除        | afterDelete  |         No          | Runs after the delete operation was made |
| 削除        | beforeDelete |         Yes         | Runs before the delete operation is made |

With the above events can also define business rules in the models:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function beforeDelete()
    {
        if ($this->status === 'A') {
            echo "The robot is active, it can't be deleted";

            return false;
        }

        return true;
    }
}
```

<a name='hydration-modes'></a>

## ハイドレーションモード

As mentioned previously, resultsets are collections of complete objects, this means that every returned result is an object representing a row in the database. These objects can be modified and saved again to persistence:

```php
<?php

use Store\Toys\Robots;

$robots = Robots::find();

// 完全なオブジェクトの結果セットを操作する
foreach ($robots as $robot) {
    $robot->year = 2000;

    $robot->save();
}
```

Sometimes records are obtained only to be presented to a user in read-only mode, in these cases it may be useful to change the way in which records are represented to facilitate their handling. The strategy used to represent objects returned in a resultset is called 'hydration mode':

```php
<?php

use Phalcon\Mvc\Model\Resultset;
use Store\Toys\Robots;

$robots = Robots::find();

// すべてのrobotを配列として返す
$robots->setHydrateMode(
    Resultset::HYDRATE_ARRAYS
);

foreach ($robots as $robot) {
    echo $robot['year'], PHP_EOL;
}

// すべてのrobotを標準クラスとして返す
$robots->setHydrateMode(
    Resultset::HYDRATE_OBJECTS
);

foreach ($robots as $robot) {
    echo $robot->year, PHP_EOL;
}

// すべてのrobotをRobotsインスタンスとして返す
$robots->setHydrateMode(
    Resultset::HYDRATE_RECORDS
);

foreach ($robots as $robot) {
    echo $robot->year, PHP_EOL;
}
```

Hydration mode can also be passed as a parameter of `find`:

```php
<?php

use Phalcon\Mvc\Model\Resultset;
use Store\Toys\Robots;

$robots = Robots::find(
    [
        'hydration' => Resultset::HYDRATE_ARRAYS,
    ]
);

foreach ($robots as $robot) {
    echo $robot['year'], PHP_EOL;
}
```

<a name='table-prefixes'></a>

## テーブル名のプレフィックス

If you want all your tables to have certain prefix and without setting source in all models you can use the `Phalcon\Mvc\Model\Manager` and the method `setModelPrefix()`:

```php
<?php

use Phalcon\Mvc\Model\Manager;
use Phalcon\Mvc\Model;

class Robots extends Model
{

}

$manager = new Manager();
$manager->setModelPrefix('wp_');
$robots = new Robots(null, null, $manager);
echo $robots->getSource(); // wp_robotsが返される
```

<a name='identity-columns'></a>

## 自動生成された id カラム

Some models may have identity columns. These columns usually are the primary key of the mapped table. [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) can recognize the identity column omitting it in the generated SQL `INSERT`, so the database system can generate an auto-generated value for it. Always after creating a record, the identity field will be registered with the value generated in the database system for it:

```php
<?php

$robot->save();

echo 'The generated id is: ', $robot->id;
```

[Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) is able to recognize the identity column. Depending on the database system, those columns may be serial columns like in PostgreSQL or auto_increment columns in the case of MySQL.

PostgreSQL uses sequences to generate auto-numeric values, by default, Phalcon tries to obtain the generated value from the sequence `table_field_seq`, for example: `robots_id_seq`, if that sequence has a different name, the `getSequenceName()` method needs to be implemented:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function getSequenceName()
    {
        return 'robots_sequence_name';
    }
}
```

<a name='skipping-columns'></a>

## カラムをスキップ

To tell [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) that always omits some fields in the creation and/or update of records in order to delegate the database system the assignation of the values by a trigger or a default:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        // INSERT / UPDATE操作の両方でフィールド/列をスキップします。
        $this->skipAttributes(
            [
                'year',
                'price',
            ]
        );

        // INSERT時のみスキップする
        $this->skipAttributesOnCreate(
            [
                'created_at',
            ]
        );

        // UPDATE時のみスキップする
        $this->skipAttributesOnUpdate(
            [
                'modified_in',
            ]
        );
    }
}
```

This will ignore globally these fields on each `INSERT`/`UPDATE` operation on the whole application. If you want to ignore different attributes on different `INSERT`/`UPDATE` operations, you can specify the second parameter (boolean) - `true` for replacement. Forcing a default value can be done as follows:

```php
<?php

use Store\Toys\Robots;

use Phalcon\Db\RawValue;

$robot = new Robots();

$robot->name       = 'Bender';
$robot->year       = 1999;
$robot->created_at = new RawValue('default');

$robot->create();
```

A callback also can be used to create a conditional assignment of automatic default values:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;
use Phalcon\Db\RawValue;

class Robots extends Model
{
    public function beforeCreate()
    {
        if ($this->price > 10000) {
            $this->type = new RawValue('default');
        }
    }
}
```

<h5 class='alert alert-warning'>Never use a <a href="api/Phalcon_Db_RawValue">Phalcon\Db\RawValue</a> to assign external data (such as user input) or variable data. The value of these fields is ignored when binding parameters to the query. So it could be used to attack the application injecting SQL. </h5>

<a name='dynamic-updates'></a>

## ダイナミックアップデート

SQL `UPDATE` statements are by default created with every column defined in the model (full all-field SQL update). You can change specific models to make dynamic updates, in this case, just the fields that had changed are used to create the final SQL statement.

In some cases this could improve the performance by reducing the traffic between the application and the database server, this specially helps when the table has blob/text fields:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->useDynamicUpdate(true);
    }
}
```

<a name='column-mapping'></a>

## 独立したカラムマッピング

The ORM supports an independent column map, which allows the developer to use different column names in the model to the ones in the table. Phalcon will recognize the new column names and will rename them accordingly to match the respective columns in the database. This is a great feature when one needs to rename fields in the database without having to worry about all the queries in the code. A change in the column map in the model will take care of the rest. 例えば:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public $code;

    public $theName;

    public $theType;

    public $theYear;

    public function columnMap()
    {
        // キーはテーブル内の実際の名前
        // 値はアプリケーション内の名前
        return [
            'id'       => 'code',
            'the_name' => 'theName',
            'the_type' => 'theType',
            'the_year' => 'theYear',
        ];
    }
}
```

Then you can use the new names naturally in your code:

```php
<?php

use Store\Toys\Robots;

// 名前でrobotを見つける
$robot = Robots::findFirst(
    "theName = 'Voltron'"
);

echo $robot->theName, "\n";

// タイプで並び替えてrobotを取得
$robot = Robots::find(
    [
        'order' => 'theType DESC',
    ]
);

foreach ($robots as $robot) {
    echo 'Code: ', $robot->code, "\n";
}

// robotを作成
$robot = new Robots();

$robot->code    = '10101';
$robot->theName = 'Bender';
$robot->theType = 'Industrial';
$robot->theYear = 2999;

$robot->save();
```

Consider the following when renaming your columns:

* リレーションやバリデータの属性への参照は、新しい名前を使用する必要があります
* 実際の列名を参照すると、ORMによる例外が発生します

The independent column map allows you to:

* 独自の規則を使用してアプリケーションを記述する
* コード内のベンダー接頭辞/接尾辞を排除する
* アプリケーションコードを変更せずに列名を変更する

<a name='record-snapshots'></a>

## レコードのスナップショット

Specific models could be set to maintain a record snapshot when they're queried. You can use this feature to implement auditing or just to know what fields are changed according to the data queried from the persistence:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->keepSnapshots(true);
    }
}
```

When activating this feature the application consumes a bit more of memory to keep track of the original values obtained from the persistence. In models that have this feature activated you can check what fields changed as follows:

```php
<?php

use Store\Toys\Robots;

// データベースからレコードを取得する
$robot = Robots::findFirst();

// 列を変更する
$robot->name = 'Other name';

var_dump($robot->getChangedFields()); // ['name']

var_dump($robot->hasChanged('name')); // true

var_dump($robot->hasChanged('type')); // false
```

Snapshots are updated on model creation/update. Using `hasUpdated()` and `getUpdatedFields()` can be used to check if fields were updated after a create/save/update but it could potentially cause problems to your application if you execute `getChangedFields()` in `afterUpdate()`, `afterSave()` or `afterCreate()`.

You can disable this functionality by using:

```php
Phalcon\Mvc\Model::setup(
    [
        'updateSnapshotOnSave' => false,
    ]
);
```

or if you prefer set this in your `php.ini`

```ini
phalcon.orm.update_snapshot_on_save = 0
```

Using this functionality will have the following effect:

```php
<?php

use Phalcon\Mvc\Model;

class User extends Model
{
  public function initialize()
  {
      $this->keepSnapshots(true);
  }
}

$user       = new User();
$user->name = 'Test User';
$user->create();
var_dump($user->getChangedFields());
$user->login = 'testuser';
var_dump($user->getChangedFields());
$user->update();
var_dump($user->getChangedFields());
```

On Phalcon 4.0.0 and later it is:

```php
array(0) {
}
array(1) {
[0]=> 
    string(5) "login"
}
array(0) {
}
```

`getUpdatedFields()` will properly return updated fields or as mentioned above you can go back to the previous behavior by setting the relevant ini value.

<a name='different-schemas'></a>

## 別のスキーマを指す

If a model is mapped to a table that is in a different schemas/databases than the default. You can use the `setSchema()` method to define that:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setSchema('toys');
    }
}
```

<a name='multiple-databases'></a>

## Setting multiple databases

In Phalcon, all models can belong to the same database connection or have an individual one. Actually, when [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) needs to connect to the database it requests the `db` service in the application's services container. You can overwrite this service setting it in the `initialize()` method:

```php
<?php

use Phalcon\Db\Adapter\Pdo\Mysql as MysqlPdo;
use Phalcon\Db\Adapter\Pdo\PostgreSQL as PostgreSQLPdo;

// このサービスはMySQLデータベースを返す
$di->set(
    'dbMysql',
    function () {
        return new MysqlPdo(
            [
                'host'     => 'localhost',
                'username' => 'root',
                'password' => 'secret',
                'dbname'   => 'invo',
            ]
        );
    }
);

// このサービスはPostgreSQLデータベースを返します
$di->set(
    'dbPostgres',
    function () {
        return new PostgreSQLPdo(
            [
                'host'     => 'localhost',
                'username' => 'postgres',
                'password' => '',
                'dbname'   => 'invo',
            ]
        );
    }
);
```

Then, in the `initialize()` method, we define the connection service for the model:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setConnectionService('dbPostgres');
    }
}
```

But Phalcon offers you more flexibility, you can define the connection that must be used to `read` and for `write`. This is specially useful to balance the load to your databases implementing a master-slave architecture:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function initialize()
    {
        $this->setReadConnectionService('dbSlave');

        $this->setWriteConnectionService('dbMaster');
    }
}
```

The ORM also provides Horizontal Sharding facilities, by allowing you to implement a 'shard' selection according to the current query conditions:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    /**
     * シャードを動的に選択
     *
     * @param array $intermediate
     * @param array $bindParams
     * @param array $bindTypes
     */
    public function selectReadConnection($intermediate, $bindParams, $bindTypes)
    {
        // select文に 'where' 句があるかどうかを確認する
        if (isset($intermediate['where'])) {
            $conditions = $intermediate['where'];

            // 条件に応じて可能なシャードを選択する
            if ($conditions['left']['name'] === 'id') {
                $id = $conditions['right']['value'];

                if ($id > 0 && $id < 10000) {
                    return $this->getDI()->get('dbShard1');
                }

                if ($id > 10000) {
                    return $this->getDI()->get('dbShard2');
                }
            }
        }

        // デフォルトのシャードを使用する
        return $this->getDI()->get('dbShard0');
    }
}
```

The `selectReadConnection()` method is called to choose the right connection, this method intercepts any new query executed:

```php
<?php

use Store\Toys\Robots;

$robot = Robots::findFirst('id = 101');
```

<a name='injecting-services-into-models'></a>

## Injecting services into Models

You may be required to access the application services within a model, the following example explains how to do that:

```php
<?php

namespace Store\Toys;

use Phalcon\Mvc\Model;

class Robots extends Model
{
    public function notSaved()
    {
        // DIコンテナからflashサービスを取得する
        $flash = $this->getDI()->getFlash();

        $messages = $this->getMessages();

        // バリデーションメッセージを表示
        foreach ($messages as $message) {
            $flash->error($message);
        }
    }
}
```

The `notSaved` event is triggered every time that a `create` or `update` action fails. So we're flashing the validation messages obtaining the `flash` service from the DI container. By doing this, we don't have to print messages after each save.

<a name='disabling-enabling-features'></a>

## 機能の有効/無効

In the ORM we have implemented a mechanism that allow you to enable/disable specific features or options globally on the fly. According to how you use the ORM you can disable that you aren't using. These options can also be temporarily disabled if required:

```php
<?php

use Phalcon\Mvc\Model;

Model::setup(
    [
        'events'         => false,
        'columnRenaming' => false,
    ]
);
```

The available options are:

| オプション                 | Description                                                        |  デフォルト  |
| --------------------- | ------------------------------------------------------------------ |:-------:|
| astCache              | すべてのモデルのコールバック、フック、イベント通知を有効/無効にする                                 | `null`  |
| cacheLevel            | ORMのキャッシュレベルを設定します。                                                |   `3`   |
| castOnHydrate         |                                                                    | `false` |
| columnRenaming        | 列名の変更を有効または無効にします。                                                 | `true`  |
| disableAssignSetters  | モデル内のセッターを無効にすることを許可する                                             | `false` |
| enableImplicitJoins   |                                                                    | `true`  |
| enableLiterals        |                                                                    | `true`  |
| escapeIdentifiers     |                                                                    | `true`  |
| events                | すべてのモデルのコールバック、フック、イベント通知を有効/無効にする                                 | `true`  |
| exceptionOnFailedSave | 失敗した `save()` が存在する場合に例外をスローするかどうかを設定します。                          | `false` |
| forceCasting          |                                                                    | `false` |
| ignoreUnknownColumns  | モデル上の未知の列の無視を有効または無効にする                                            | `false` |
| lateStateBinding      | `Phalcon\Mvc\Model::cloneResultMap()` メソッドの遅延バインディングを有効または無効にします | `false` |
| notNullValidations    | ORMは、マップされた表に存在するNOT NULL列を自動的に検証します                               | `true`  |
| parserCache           |                                                                    | `null`  |
| phqlLiterals          | PHQLパーサのリテラルを有効/無効にする                                              | `true`  |
| uniqueCacheId         |                                                                    |   `3`   |
| updateSnapshotOnSave  | `save()` でスナップショットの更新を有効または無効にします。                                 | `true`  |
| virtualForeignKeys    | 仮想外部キーを有効/無効にします。                                                  | `true`  |

<div class="alert alert-warning">
    <p>
        <strong>備考</strong> <code>Phalcon\Mvc\Model::assign()</code> （モデルの作成/更新/保存時にも使用されます）は、データ引数が渡されたときには常にセッターが使われます。 これにより、アプリケーションにオーバーヘッドが追加されます。 この動作を変更するには、 <code>phalcon.orm.disable_assign_setters = 1</code> をiniファイルに追加します。これは単に <code>$this->property = value</code> を使用するだけです。
    </p>
</div>

<a name='stand-alone-component'></a>

## 独立コンポーネント

Using [Phalcon\Mvc\Model](api/Phalcon_Mvc_Model) in a stand-alone mode can be demonstrated below:

```php
<?php

use Phalcon\Di;
use Phalcon\Mvc\Model;
use Phalcon\Mvc\Model\Manager as ModelsManager;
use Phalcon\Db\Adapter\Pdo\Sqlite as Connection;
use Phalcon\Mvc\Model\Metadata\Memory as MetaData;

$di = new Di();

// コネクションの初期化
$di->set(
    'db',
    new Connection(
        [
            'dbname' => 'sample.db',
        ]
    )
);

// モデルマネージャを設定する
$di->set(
    'modelsManager',
    new ModelsManager()
);

// メタデータ記憶アダプターなどを使用する
$di->set(
    'modelsMetadata',
    new MetaData()
);

// モデルを作成する
class Robots extends Model
{

}

// モデルを使用する
echo Robots::count();
```
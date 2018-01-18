<!--
Categories = ["Development", "Laravel"]
Description = "Extending 'hasManyIn' relationship for Eloquent Model"
Tags = ["Development", "laravel", "hasMany"]
date = "2018-01-17T21:47:31-08:00"
title = "Extending 'hasManyIn' relationship for Eloquent Model"
-->

## Extending 'hasManyIn' relationship for Eloquent Model

Recently we need use Laravel to replace some exist portals. Because we need to run Laravel and old portal parallelly. So we can not modify database's structure. 

There're some exists tables manage relationship by using text field contain some ids like this: **'id1,id2,id3,id4'**, seperates with comma.
It's not a good design, but we have no choice to inherit the legacy.

Here's an example of two tables relationship.

TableA has many TableB's record.

```shell
TableA    
field: id, name,     tableb_ids
rows:  1,  ta_name1,  1,2,3,4
       2,  ta_name1,  5,6,7,8

TableB
field: id, name
rows:  1,  tb_name1
       2,  tb_name2
       3,  tb_name3
       4,  tb_name4
       5,  tb_name5
       6,  tb_name6
       7,  tb_name7
       8,  tb_name8
```

#### 1. Class HasManyIn

```php
<?php

namespace Yong\Eloquent\Model;

use Illuminate\Database\Eloquent\Collection;

class HasManyIn extends \Illuminate\Database\Eloquent\Relations\HasMany
{
    public function getResults()
    {
        $this->addEagerConstraints([$this->parent]);
        return $this->query->get();
    }

    public function addConstraints()
    {
        if (static::$constraints) {
            $this->query->whereNotNull($this->foreignKey);
        }
    }

    public function addEagerConstraints(array $models) {
        $data = [];
        $keys = $this->getKeys($models, $this->localKey);
        foreach ($keys as $key) {
            $data = array_merge($data, explode(",", $key));
        }

        $this->query->whereIn($this->foreignKey, array_unique(array_map('trim', $data)));
    }

    function matchOneOrMany(array $models, Collection $results, $relation, $type) {
        $dictionary = $this->buildDictionary($results);
        foreach ($models as $model) {
            $keys = array_unique(array_map('trim', explode(",", $model->getAttribute($this->localKey))));
            $data = [];
            foreach ($keys as $key) {
                if (isset($dictionary[$key])) {
                    $data[] = $dictionary[$key][0];
                }
            }
            if (count($data)) {
                $model->setRelation($relation, $this->related->newCollection($data));
            }
        }
        return $models;
    }
}
```
#### 2. trait TraitHasManyIn

```php
<?php

namespace Yong\Eloquent\Model;

use Closure;
use Illuminate\Database\Eloquent\Relations\HasOne;
use Yong\Eloquent\Model\HasManyIn;

trait TraitHasManyIn
{
    public function hasManyIn($related, $foreignKey = null, $localKey = null) {
        $foreignKey = $foreignKey ?: $this->getForeignKey();

        $instance = $this->newRelatedInstance($related);

        $localKey = $localKey ?: $this->getKeyName();

        return new HasManyIn($instance->newQuery(), $this, $instance->getTable() . '.' . $foreignKey, $localKey);
    }

    protected function newRelatedInstance($class)
    {
        if ($class instanceof Closure) {
            return tap($class->call($this), function ($instance) {
                if (! $instance->getConnectionName()) {
                    $instance->setConnection($this->connection);
                }
            });
        }
        return parent::newRelatedInstance($class);
    }
}
```

### How to use

1. Your Eloquent Models

For TableA
```php
namespace YourNamespace;

class TableA extends \Illuminate\Database\Eloquent\Model
{
    use \Yong\Eloquent\Model\TraitHasManyIn;   //use this trait

    public function table_b_records() {
        return $this->hasManyIn(TableB::class, 
            'id',           //TableB's primary key 
            'tableb_ids'    //TableA's relationship key field
        );
    }
}
```
For TableB

```php
namespace YourNamespace;

class TableB extends \Illuminate\Database\Eloquent\Model
{
    
}
```

2. Get hasManyIn relationship

```php

$insA = TableA::find(1);
$conllection = $insA->table_b_records;
//it's the collection which contains model TableB instances(id in (1,2,3,4))
```
# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 3 : Implementing a table in small-orm model system

### Model system

small-orm is based on DAO design pattern.

To acces database, you have a DAO object. This DAO transform data in objects (models).

In addition we have Validators to validate models before persist.

Then we have three types of objects :
* DAO
* Model
* Validator

### Creating a DAO

For each database table, we have a DAO class. This class must extends 'Sebk\SmallOrmCore\Dao\AbstractDao' abstract class.

The goal of this class is to define the bridge between database and model.

The first step is to define the structure of model and how to fill it with data. To do than, you must define a 'build' method.

Here is an example :
```injectablephp
<?php

namespace App\Model\TestBundle\Dao;

use Sebk\SmallOrmCore\Dao\AbstractDao;

class Customer extends AbstractDao
{

    protected function build()
    {
        $this->setDbTableName("customer")
            ->setModelName("Customer")
            ->addField("id", "id")
            ->addField("email", "email")
            ->addField("firstname", "firstname")
            ->addField("lastname", "lastname")
            ->addField("id_type", "idType")
            ->addToOne("type", ["idType" => "id"], "CustomerType")
            ->addToMany("orders", ["id" => "idCustomer"], "Order")
        ;
    }

}
```

* 'setDbTableName' : define the name of database table
* 'setModel' define the model class corresponding to the DAO. By convention, it have the same name but you can change it if you want.
* 'addField' : link a field database name with a model property name
* 'addToOne' : define a model property of model to link with another model. 
  * Here the property 'type' of model 'Customer' will point to an 'CustomerType' model.
  * The query builder will use it in order to join in sql query.
  * The join materialization in sql is based on the second parameter : the field 'id' of model 'Customer' will be used to join on the field 'idCustomer' of the model 'CustomerType'
* 'addToMany' : define a group of models to link with the model of DAO as the same way as 'toOne' parameter

By convention, the DAO is also, a repository of requests (see [Chapter 4 - QueryBuilder](chapter-4.md) and [Chapter 5 - RedisQueryBuilder](chapter-5.md))

Now, you can get a DAO objet with 'sebk_small_orm_dao' factory service.

For Symfony :
```injectablephp
$daoCustomer = $container->get('sebk_small_orm_dao')->get('TestBundle', 'Customer');
```

For Swoft :
```injectablephp
$daoCustomer = bean('sebk_small_orm_dao')->get('TestBundle', 'Customer');
```

## Creating models definition and use them

The models class define objects that will be return by DAO in result of queries. Often, ORM call them entities.

Here is the minimum definition of model our 'Customer' :
```injectablephp
<?php

namespace App\Model\TestBundle\Model;

use Sebk\SmallOrmCore\Dao\Model;

class Customer extends Model
{
}
```

The only required thing is that your model class extends 'Sebk\SmallOrmCore\Dao\Model'.

When queried, the DAO will create model with getters and setters corresponding to the definition in 'build' of DAO :

```injectablephp
$idType = $customer->getIdType();
$customer->setFirstName('John');
```

You can also create a new empty model from dao :
```injectablephp
$newCustomer = $daoCustomer->newModel();
```

In addition, you can add custom methods. For example, we will calculate the amount of spent money by customer :

```injectablephp
<?php

namespace App\Model\TestBundle\Model;

use Sebk\SmallOrmCore\Dao\Model;

class Customer extends Model
{
    public function getTotalSpent(): float
    {
        $totalCustomer = 0;
        foreach($this-getOrders() as $order) {
            $totalCustomer = $order->getTotal();
        }
        
        return $totalCustomer;
    }
}
```

You can now use this method in all your models :
```injectablephp
$totalCustomer = $customer->getTotalSpent();
```

You can also set 'on fly' properties :
```injectablephp
$customer->setMyProp("test");
```

And get them later :
```injectablephp
$result = $customer->getMyProp();
```

There is some 'special' methods that help in some cases :
```injectablephp
<?php

namespace App\Model\TestBundle\Model;

use Sebk\SmallOrmCore\Dao\Model;

class Customer extends Model
{
    public function onLoad()
    {
        echo "loaded";
    }

    public function beforeSave()
    {
        echo "prepare persist";
    }

    public function afterSave()
    {
        echo "Persist done";
    }

    public function beforeDelete()
    {
        echo "Prepare delete";
    }

    public function afterDelete()
    {
        echo "Delete done";
    }
}
```

These methods are called on action on model :
* onLoad : called when the model is built from database (not with 'newModel' method of DAO)
* beforeSave : called just before saving model to database
* afterSave : called just after saving model to database
* beforeDelete : called just before deleting a model in database
* afterSave : called just after deleting a model in database

### Interact with database

To save model (insert or update), just call 'perist' method :
```injectablephp
$customer->setName('John');
$customer->persist();
```

Or with DAO object :
```injectablephp
$customer->setName('John');
$customerDao->persist($customer);
```

These two ways do exactly the same.

As same way, you can delete a model :
```injectablephp
$customer->delete();
```

To load a model from database, you can use 'findOne' shortcut :
```injectablephp
$customer = $daoCustomer->findOneBy(['id' => 1]);
```

This request will build '$customer' model and load it from database with customer id 1.

At this point, we want to read label of customer type.
```injectablephp
$type = $customer->getType();
```

As we have no lasy loading, $type is null.

There is two ways to load the dependency :
* In the same request :
```injectablephp
$customer = $daoCustomer->findOneBy(['id' => 1], [['type']]);
$type = $customer->getType();
```

Here, the $type will contains the 'CustomerType' model corresponding to customer id 1.

You can get all levels of dependency as you want. Here another example that load customer and count the number of items of he's first order
```injectablephp
$customer = $daoCustomer->findOneBy(['id' => 1], [['orders'], ['orders' => 'items']]);
$number = count($customer->getOrders()[0]->getItems());
```

* With loadToOne and loadToMany methods of model
```injectablephp
// Load customer
$customer = $daoCustomer->findOneBy(['id' => 1]);

// Load and get type
$customer->loadToOne('type');
$type = $customer->getType();

// Load and get orders
$customer->loadToMany('orders');
$orders = $customer->getOrders();
```

Note that if the type or orders have been already set (manually or from database), the loadToOne and loadToMany does nothing (no interactions with database).

The good way to load dependency depends on your process.
* The first way is faster if you are sure to use all dependencies
* The second way is safer for reusable functions because if already loaded, it changes nothing, and if not you are sure to have the dependency loaded for your process

To load many models, use the 'findBy' method :
```injectablephp
$customers = $customerDao->findBy(['idType' => 2]);
```

$customers will load from database an array of all customers of type id 2.

For more complex requests, see [chapter 4 - QueryBuilder](chapter-4.md).

### Redis specials

Redis is a key/value database.

Then the 'findOneBy' and 'findBy' methods is more simple to use :
```injectablephp
$value = $daoRedis->findOneBy(2); // Load key my:key:2
$values = $daoRedis->findBy([2, 3, 4]); // Load keys my:key:2, my:key:3 and my:key:4 as array of models
$value = $daoRedis2->findOneBy(); // Load key my:second-key
```
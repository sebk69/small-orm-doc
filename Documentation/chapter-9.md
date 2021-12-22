# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 9 : Working with types

### Types in small-orm

small-orm has type management in ORM side only. That mean that your fields model types can be different than database types.

For example, a database field INT can be declared TIMESTAMP in your DAO.

Types are used by some automatic conversions processes between your applications layers (front/api/database) and for automatic forms validations.

### DAO declaration

We have seen the 'addField' method to initialize you DAO. There are some optionals parameters we don't talk about. Here is a more advanced Customer DAO :
```php
<?php

namespace App\Model\TestBundle\Dao;

use Sebk\SmallOrmCore\Dao\AbstractDao;

class Customer extends AbstractDao
{

    protected function build()
    {
        $this->setDbTableName('customer')
            ->setModelName('Customer')
            ->addPrimaryKey('id', 'id')
            ->addField('email', 'email', null, Field::TYPE_STRING)
            ->addField('firstname', 'firstname')
            ->addField('lastname', 'lastname')
            ->addField('id_type', 'idType', null, Field::TYPE_INT)
            ->addField('profit', 'profit', 0, Field::TYPE_FLOAT)
            ->addField('validated', 'validated', false, Field::TYPE_BOOLEAN, ['off', 'on'])
            ->addField('created_at', 'createdAt', null, Field::TYPE_DATETIME, 'd/m/Y h:i:s')
            ->addField('last_visit_time', 'lastVisitTime', null, Field::TYPE_TIMESTAMP, 'd/m/Y h:i:s')
            ->addToOne('type', ['idType' => 'id'], 'CustomerType')
            ->addToMany('orders', ['id' => 'idCustomer'], 'Order')
        ;
    }

}
```

Let see now the new parameters :
* The third parameter is the default value. When a new model is created by 'newModel' method of DAO, this value is filled in model.
* The fourth parameter is the type. If none specified (or null), the type will be STRING.
* The fifth parameter is used only by DATETIME, TIMESTAMP and BOOLEAN type, it defines behaviour of the format

Here are type explanations :
* The Field::TYPE_STRING do nothing : it is managed as you haven't specified type.
* The Field::TYPE_INT : when json_encode on model, the field will be considered to number. Useful for a typescript front for examples.
* The Field::TYPE_FLOAT : identical than Field::TYPE_INT, except it accept decimal numbers
* The Field::TYPE_BOOLEAN : when json_encode on model, the field will be considered to boolean. When persisted, the format specified will be used to convert boolean in database value. In our example, the database value will be 'off' when ORM value is false and the database value will be 'on' when ORM value is true. In the same way, the ORM value will be true on load when database value is 'on' and false in other cases.
* The Field::TYPE_DATETIME : when json_encode on model, the field will have the specified format. When persist in database, a 'Y-m-d H:i:s' format will be used. In ORM the field will have a \DateTime type.
* The Field::TYPE_TIMESTAMP have the same behaviour as Field::TYPE_DATETIME, except that the database format will be INT unix timestamp. Useful for legacy database tables for example. 
* The Field::TYPE_PHP_FILTER : is exactly same as Field::TYPE_STRING but when imported to forms, allow you to check format as in 'filter_var' php function. The format must be one of the filter of 'filter_var' (see [https://www.php.net/manual/en/filter.filters.validate.php](https://www.php.net/manual/en/filter.filters.validate.php))

In addition of these behaviours, the type have a very important place in small-orm-form package (see [chapter 10](chapter-11.md)).
# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 4 : QueryBuilder

### What is QueryBuilder ?

QueryBuilder is an helper classes system that help you build complex queries to database.

At final, it will generate a sql query that can be executed which result can be interpreted by DAO to build models as response.

By convention, we will put all our queries in your DAO class to keep them in one place as a repository.

### Basic operations

#### Instanciate query

To build a query, just call 'createQueryBuilder' method of DAO :
```php
$query = $daoCustomer->createQueryBuilder();
```

To see corresponding sql (usefull for debugging), you can call 'getSql' method of QueryBuilder :
```php
echo $query->getSql();
```

In output, you will see this sql :
```sql
SELECT DISTINCT `Customer`.`id` AS `Customer_id`, `Customer`.`email` AS `Customer_email`, `Customer`.`firstname` AS `Customer_firstname`, `Customer`.`lastname` AS `Customer_lastname`, `Customer`.`id_type` AS `Customer_idType` FROM `customer` AS `Customer`
```

#### Get query result

To get result, use 'getResult' method of DAO :
```php
$allCustomers = $daoCustomer->getResult($query);
```

#### Put it in DAO

As we say previously, we want to keep all our requests in DAO. Here is our DAO code :
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
            ->addField('id', 'id')
            ->addField('email', 'email')
            ->addField('firstname', 'firstname')
            ->addField('lastname', 'lastname')
            ->addField('id_type', 'idType')
            ->addToOne('type', ['idType' => 'id'], 'CustomerType')
            ->addToMany('orders', ['id' => 'idCustomer'], 'Order')
        ;
    }

    /**
     * Get all customers
     * @return \App\Model\TestBundle\Model\Customer[]
     * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
     */
    public function getAllCustomers(): array
    {
        $query = $this->createQueryBuilder();

        return $this->getResult($query);
    }
}
```

#### Where clause

The where clause allow you to filter your results. It comes with conditions instructions.

For example, we want to get all customers of type id 1 :
```php
$query->where()
    ->firstCondition($query->getFieldForCondition('idType'), '=', 1);
```

The where clause come always with a least one condition using 'firstCondition' method.

Condition take 3 parameters :
* A field, a value or a binding
* An operator
* A field, a value or a binding

A field is given by 'getFieldForCondition' method. Here we have taken 'idType' field.

In result, we have the following 'getSql' result :
```sql
SELECT DISTINCT `Customer`.`id` AS `Customer_id`, `Customer`.`email` AS `Customer_email`, `Customer`.`firstname` AS `Customer_firstname`, `Customer`.`lastname` AS `Customer_lastname`, `Customer`.`id_type` AS `Customer_idType` FROM `customer` AS `Customer`  WHERE  `Customer`.`id_type` = '1'
```

Now we want to build a 'getCustomerByType' injecting a parameter to generalise the request. To do that, we use binding with the 'setParameter' method :
```php
/**
 * Get customer by type
 * @param int $idType
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\BracketException
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getCusomersByType(int $idType): array
{
    $query = $this->createQueryBuilder();
    
    $query->where()
        ->firstCondition($query->getFieldForCondition('idType'), '=', ':idType')
    ;
    
    $query->setParameter('idType', $idType);

    return $this->getResult($query);
}
```

#### Operators

All Mysql operators are implemented :
* '='
* '<'
* '>'
* '<='
* '>='
* '!='
* '<>'
* '%'
* 'like'
* 'not like'
* 'not regexpr'
* 'regexpr'
* 'is'
* 'is not'
* 'exists'
* 'not exists'
* 'in'
* 'not in'

### Join models

Now, we want to get customers with customer types and orders.

To do that, we use join methods on 'toOne' and 'toMany' fields. Here is now our 'getAllCustomers' request :
```php
/**
 * Get all customers
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getAllCustomers(): array
{
    $query = $this->createQueryBuilder('customer')
        ->innerJoin('type')->endJoin()
        ->leftJoin('orders')->endJoin()
    ;

    return $this->getResult($query);
}
```

This request will load 'type' and 'orders' properties with corresponding models in one request to database :
```php
$customers = $daoCustomer->getAllCustomers();
echo $customers[0]->getType()->getLabel();
foreach ($customers[0]->getOrders() as $order) {
    echo $order->getTotal();
}
```

Here are the possible join methods :
* 'join' ('innerJoin' shortcut)
* 'innerJoin'
* 'leftJoin'
* 'rigthJoin'
* 'fullOuterJoin'

You can also use conditions in joins. For example, we want to filter orders by status 'prepared' of customers :
```php
/**
 * Get all customers with prepared orders
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getCustomersOrdersPrepared(): array
{
    $query = $this->createQueryBuilder('customer');
    $query->innerJoin('type')->endJoin()
        ->leftJoin('orders')
            ->joinCondition()
                ->andCondition($query->getFieldForCondition('status', 'orders'), '=', 'prepared')
            ->endJoinCondition()
        ->endJoin()
    ;

    return $this->getResult($query);
}
```

### Multiple conditions

Of course you can use more than one condition in 'where' clause of in 'joinCondition' clause :
```php
$query = $daoCustomer->createQueryBuilder('customer');
$query->innerJoin('type')->endJoin()
    ->leftJoin('orders')
        ->joinCondition()
            ->andCondition($query->getFieldForCondition('status', 'orders'), '=', 'prepared')
            ->andCondition($query->getFieldForCondition('total', 'orders'), '>', 100)
        ->endJoinCondition()
    ->endJoin()
;
$query->where()
    ->firstCondition($query->getFieldForCondition('idType'), '=', '1')
    ->andCondition($query->getFieldForCondition('firstname'), 'like', 'john%')
;
```

In this request, we get all customers of type id 1, which firstname begin by 'john' and get only prepared orders of more than 100$.

Here are possible conditions :
* 'firstCondition'
* 'andCondition'
* 'orCondition'
* 'xorCondition'

### Brackets

To continue in complex requests, you can use brackets on your 'where' clause or 'joinCondition' clause :
```php
$query = $daoOrder->createQueryBuilder('order');
$query->where()
    ->firstCondition($query->getFieldForCondition('total'), '>', 100)
    ->andCondition($query->getFieldForCondition('total'), '<', 1000)
    ->andBracket()
        ->firstCondition($query->getFieldForCondition('status'), '=', 'prepared')
        ->orCondition($query->getFieldForCondition('status'), '=', 'in progress')
    ->endBracket()
    ->andCondition($query->getFieldForCondition('idTax'), '=', 5)
```

This request will return all orders between 100$ and 1000$, prepared or in progess which tax id is 5.

Note that the first condition of bracket must be 'firstCondition'.

(TODO sub queries)
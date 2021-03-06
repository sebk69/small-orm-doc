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
    ->firstCondition($query->getFieldForCondition('idType'), '=', 1)
;
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
;
```

This request will return all orders between 100$ and 1000$, prepared or in progess which tax id is 5.

Note that the first condition of bracket must be 'firstCondition'.

### Sub queries

You can also create sub queries.

The 'getFieldForCondition' method work even it the target query is not the from query.

Let see an example : You want to list customers that have at least an order prepared.

Let start making sub query to list prepared orders :

```php
$subquery = $daoOrder->createQueryBuilder('order');
$subquery->where()
    ->firstCondition($query->getFieldForCondition('status'), '=', 'prepared')
;
```

Now we build the customer (main) query :

```php
$query = $daoCustomer->createQueryBuilder('customer');
$query->where()
    ->firstCondition($subquery, 'exists')
;
```

And complete the sub query with cross query condition :
```php
$subquery->getWhere()
    ->andCondition($subquery->getFieldForCondition('idCustomer'), '=', $query->getFieldForCondition('id'))
;
```

### Pagination

If you want to limit results, you can use the 'paginate' method.

```php
/**
 * Get customers by page
 * @param $page
 * @param int $pageSize
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getCustomers($page, $pageSize = 10): array
{
    $query = $this->createQueryBuilder('customer');
    
    $query->paginate($page, $pageSize);

    return $this->getResult($query);
}
```

The first parameter is page number and the second parameter is the number of result in each pages.

### Order by

You can order the result by one or more fields :
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
    
    $query->addOrderBy('email', null, 'ASC');
    $query->addOrderBy('date', 'orders', 'DESC');

    return $this->getResult($query);
}
```

* The first parameter is the field.
* The second parameter is the model of field
* The third is 'ASC' or 'DEC' corresponding to the order sens of sort

By default (if no order by is specified), the result is sorted by primary keys.

In this example, the result will be sorted by email (ascending) and in the 'orders' dependence by 'date' (descending).

### Raw operations

In some cases, you will need to do requests outside object oriented scope. For example, you want to do a count or a sum. The raw operations are here to do that.

#### Raw select

You can use 'rawSelect' method of QueryBuilder in order to get result as array like you have done if you have used pdo :
```php
/**
 * List emails for customers of a type
 * @param $idType
 * @return array
 * @throws \Sebk\SmallOrmCore\QueryBuilder\BracketException
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getEmailsForType($idType)
{
    $query = $this->createQueryBuilder('customer');
    
    $query->rawSelect('customer.firstname as firstname, customer.lastname as lastname, customer.email as email');
    
    $query->where()
        ->firstCondition($query->getFieldForCondition('idType'), '=', ':idType')
    ;
    
    $query->setParameter('idType', $idType);
    
    return $this->getResult($query);
}
```

You must specify the clause as you've typed in sql format with db field names instead of model field name.

The table alias is the string you have put in 'createQueryBuilder' method call.

The result will be an array of record. Each record key if the col name of result :
```json
[
  {"fisrtname": "john", "lastname": "doe", "email": "john.doe@mymail.com"},
  {"firstname": "paul", "lastname": "fils", "email": "P.f@nomail.net"}
]
```

#### Raw where

You can use 'rawWhere' in order add conditions to your queries in sql format :
```php
/**
 * List emails for customers of a type
 * @param $idType
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\BracketException
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function getEmailsForType($idType)
{
    $query = $this->createQueryBuilder('customer');
        
    $query->setRawWhere('customer.id_type = :idType');
    
    $query->setParameter('idType', $idType);
    
    return $this->getResult($query);
}
```

As we don't use 'rawSelect', the result should be an array of models.

You must specify the clause as you've typed in sql format with db field names instead of model field name.

The table alias is the string you have put in 'createQueryBuilder' method call.

#### Raw order by

The method 'setRawOrderBy' allow you to create 'orderby' clause as you've typed in sql :
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
    
    $query->setRawOrderBy('cusotmers.email, orders.date DESC');

    return $this->getResult($query);
}
```

In this example, the result will be sorted by email (ascending) and in the 'orders' dependence by 'date' (descending).

#### Raw group by

You can do 'group by' in your requests with the 'rawGroupBy' method :
```php
/**
 * List types and count number of cutomers
 * @return \App\Model\TestBundle\Model\Customer[]
 * @throws \Sebk\SmallOrmCore\QueryBuilder\BracketException
 * @throws \Sebk\SmallOrmCore\QueryBuilder\QueryBuilderException
 */
public function countTypes()
{
    $query = $this->createQueryBuilder('customer');
        
    $query->rawSelect('customer.id_type idType, count(*) numCustomers')
    
    $query->setRawGroupBy('customer.id_type');
    
    return $this->getResult($query);
}
```

The result of this example will something like :
```json
[
  {"idType": 1, "numCustomers": 356},
  {"numCustomers": 2, "numCustomers":  1523}
]
```
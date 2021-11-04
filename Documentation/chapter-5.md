# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 5 : RedisQueryBuilder

Of course, redis is a nosql database and you can't use sql QueryBuilder.

Redis use a specific QueryBuilder.

### Create query

The query creation is exactly the same as QueryBuilder :
```php
$query = $daoRefundCustomer->createQueryBuilder();
```

### Get a key

To get a key, just use 'get' method :
```php
$query->get(1);
```

You can chain this clause :
```php
$query
    ->get(1)
    ->get(2)
    ->get(3)
;
```

To get result, as same as QueryBuilder, use 'getResult' method of you DAO :
```php
$query = $daoRefundCustomer->createQueryBuilder();
$query
    ->get(1)
    ->get(2)
    ->get(3)
;
$refunds = $daoRefundCustomer->getResult($query);
```

Note that when you chain 'get' clause, the 'mget' redis instruction is used to get keys in one tcp request.

### Set a key

To set a key, use 'set' method :
```php
$query->set(1, $refund);
```

As 'get' method, you can chain 'set' :
```php
$query
    ->set(1, $refunds[0])
    ->set(2, $refunds[1])
    ->set(3, $refunds[2])
;
```

### Delete a key

To delete a key, use 'del' method :
```php
$query
    ->del(1)
    ->del(2)
;
```
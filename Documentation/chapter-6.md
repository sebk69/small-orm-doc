# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 6 : UpdateBuilder and DeleteBuilder

### Doing mass update and mass delete are not compatible object philosophy... Why in an ORM ?

Doing an update or delete on many recors can be very slow using models.

To avoid that situation, come UpdateBuilder and DeleteBuilder. These classes works the same as QueryBuilder but generated sql is 'update' or 'delete' sql instructions.

Be careful : the model validator and the specials 'beforeSave' and 'afterSave' methods are not called on mass update. In the same way, the 'beforeDelete' and the 'afterDelete' methods are not called on mass delete.

### UpdateBuilder

#### Create query

To create a mass update query, use the 'createUpdateBuilder' method of your DAO and add the fields to update :
```php
$query = $daoCustomer->createUpdateBuilder()
    ->addFieldToUpdate('idType', 5)
    ->addFieldToUpdate('firtname', 'paul')
;
```

For now, this query will update all customers, setting type id to 5 and firstname to 'paul'

#### Execute query

To execute mass update query, use 'executeUpdate' method of DAO :
```php
$daoCustomer->executeUpdate($query);
```

#### Add where clause

You can add where clause to your update query. The where clause of an update query is strictly the same as for select QueryBuilder :
```php
$query = $daoCustomer->createUpdateBuilder()
    ->addFieldToUpdate('idType', 5)
    ->addFieldToUpdate('firtname', 'paul')
;
$query->where()
    ->firstCondition($query->getFieldForCondition('firstname', '=', 'john')
;
```

This example query will update all customers which firstname is 'john'.

#### Bind parameters

To bind parameters to an update query, just do the same as for select QueryBuilder :
```php
$query = $daoCustomer->createUpdateBuilder()
    ->addFieldToUpdate('idType', ':idType')
    ->addFieldToUpdate('firtname', ':finalFirstname')
;
$query->where()
    ->firstCondition($query->getFieldForCondition('firstname', '=', ':forFirstname')
;
$query->setParameter('idType', 5)
    ->setParameter('finalFirstname', 'paul')
    ->setParameter('forFirstname', 'john')
;
```

### Deletebuilder

To create a mass delete query, use the 'createDeleteBuilder' method of your DAO :
```php
$query = $daoCustomer=>createDeleteBuidler();
```

#### Execute query

To execute mass delete, use 'executeDelete' method :
```php
$daoCustomer->executeDelete($query);
```

#### Add where clause

You can add where clause to your delete query, same as QueryBuilder and UpdateBuilder :
```php
$query = $daoCustomer->createDeleteBuilder();
$query->where()
    ->firstCondition($query->getFieldForCondition('firstname', '=', 'john')
;
```

#### Bind parameter

To bind parameters, the process is the same as usual :
```php
$query = $daoCustomer->createDeleteBuilder();
$query->where()
    ->firstCondition($query->getFieldForCondition('firstname', '=', ':firstname')
;
$query->setParameter('firstname', $firstname);
```
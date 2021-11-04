# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 2 : Installation and configuration for Symfony

This section is for Symfony 5+ applications

### Install small-orm on existing Symfony application

Go to root path of your Symfony application.

Require the core package with composer :
```bash
composer require sebk/small-orm-core "1.*"
```

Require the bundle package with composer :
```bash
composer require sebk/small-orm-bundle "~2.0"
```

Register bundle in config/bundles.php :
```php
<?php

return [
    ...
    Sebk\SmallOrmBundle\SebkSmallOrmBundle::class => ['all' => true],
    ...
];
```

### Configuring small-orm-bundle in your application

```yaml
sebk_small_orm:
    connections:
        default:
            type: mysql
            host: %db_host%
            database: %database_name%
            encoding: utf8
            user:     %db_user%
            password: %db_password%

    bundles:
        AppAcmeBundle:
            connections:
                default:
                    dao_namespace: App\AcmeBundle\Dao
                    model_namespace: App\AcmeBundle\Model
                    validator_namespace: App\AcmeBundle\Validator
```

A bundle must have following subfolders/namespaces part :
* _**Dao**_ : This folder contains all your DAO classes (used to access database)
* _**Model**_ : This folder contains all your models classes (equivalent of entities)
* _**Validator**_ : This folder contains all your validators classes (classes used to validate integrity of your models before persist)

You must have at least one 'default' connection.

For now, only mysql type is implented. This is planned to implement a Redis and MongoDb connectors in the future.
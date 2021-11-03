# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 1 : Installation and configuration for Swoft

### Install small-orm on existing Swoft application

Go to root folder of your application.

Require Small ORM Core package :
```
composer require sebk/small-orm-core
```

Require the swoft package with composer :
```
composer require sebk/small-orm-swoft
```

To use CRUD generator (this step is optional), you must require sebk/swoft-json-response and sebk/small-orm-forms :
```
composer require sebk/swoft-json-response
composer require sebk/small-orm-forms
```

### Configuring small-orm-swoft in your application

Here is the code to put in your config/sebk_small_orm.php :
```
return [
    'bundlesBasePath' => __DIR__ . '/../app/Bundles/',
    'crudBasePath' => __DIR__ . '/../app/Http/Controller/',
    'connections' => [
        'default' => [
            'type' => 'swoft-mysql',
            'host' => 'localhost',
            'database' => 'swoft_db',
            'encoding' => 'utf8',
            'user' => 'root',
            'password' => 'dev',
            'tryCreateDatabase' => false,
        ],
    ],
    'bundles' => [
        'TestModel' => [
            'connections' => [
                'default' => [
                    'dao_namespace' => 'App\Bundles\TestModel\Dao',
                    'model_namespace' => 'App\Bundles\TestModel\Model',
                    'validator_namespace' => 'App\Bundles\TestModel\Validator',
                ],
            ],
        ],
    ],
];
```

The 'bundleBasePath' is the root path of your bundles. Each bundle is a subfolder in this path.

A bundle must have following subfolders/namespaces part :
* _**Dao**_ : This folder contains all your DAO classes (used to access database)
* _**Model**_ : This folder contains all your models classes (equivalent of entities)
* _**Validator**_ : This folder contains all your validators classes (classes used to validate integrity of your models before persist)

The 'crudBasePath' is your controllers folder. Automatic crud generation will create controllers here.

You must have at least one 'default' connection.

The following types are allowed for now :
* swoft-mysql : MySql connector for swoft (using default async pool of Swoft)
* swoft-redis : Redis connector for swoft (using default async pool of Swoft)
* mysql : traditionnal mysql connector. Avoid to use it in Swoft applications, because it don't support async programming and connection can fail if not used for a moment.

If the tryCreateDatabase is true, the database will be created automatically if not exists. It can slow your app, then use it only for dev purpose.

In 'bundles' section, you can define as many bundles you want. Here we have created a TestModel bundle that can be used by 'default' connection (it is possible to define bundles handle more that one connection).

In connection section, is the definition of namespaces of each subfolders of bundle.
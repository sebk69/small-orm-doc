# small-orm-doc

small-orm-doc is the documentation for small-orm project

You can read description of small-orm project bellow, or direct access [Documentation](Documentation/table-of-content.md)

## What is small-orm ?

As small-orm is an ecosystem of packages serving a simple but efficient ORM for php developpers.

It started from considering that lazy loading is not a good practice in terms of performance.

At each sub-object reached with lazy loading, a request is done, including network latence and time for database engine to build request.

small-orm allow you to load a group of objects in one time with joins, which is a heavy gain of performance for complex databases.

The around packages are oriented to api and microservice.

## Which databases engines supported ?

For now, Mysql and Redis are supported. It is planned to implement MongoDb.

## What about frameworks integration ?

small-orm is implemented in two frameworks :
* [Symfony](https://symfony.com)
* [Swoft](http://swoft.io)

It has been developed originally for Symfony but the recent efforts are on Swoft.

## Packages

### small-orm-core

This is the base package for small-orm. If your project is a non framework project or in a non implemented framework, use this package.

You can take it via :
* composer : https://packagist.org/packages/sebk/small-orm-core
* github : https://github.com/sebk69/small-orm-core

### small-orm-bundle

This is a Symfony bundle to easy use small-orm in Symfony.

Before 2.x versions, it not needs small-orm-core : the core was originally developped in this bundle.

Since 2.x versions, the core have been extracted in order to open small-orm to other frameworks.

Note : for now the redis connector is not yet implemented

You can take it via :
* composer : https://packagist.org/packages/sebk/small-orm-bundle
* github : https://github.com/sebk69/SebkSmallOrmBundle

### small-orm-swoole

This is a swoole connector for small-orm.

You can take it via :
* composer : https://packagist.org/packages/sebk/small-orm-swoole
* github : https://github.com/sebk69/small-orm-swoole

A symfony sandbox is available
* github : https://github.com/sebk69/small-orm-swoole-sandbox

### small-orm-swoft

This is a Swoft package for small-orm.

This package come with specific connectors (MySql and Redis) in order to manage async requests.

You can take it via :
* composer : https://packagist.org/packages/sebk/small-orm-swoft
* github : https://github.com/sebk69/small-orm-swoft

### small-orm-forms

This is a cross framework package. The goal of this package is to easily validate api calls and patch models.

You can take it via :
* composer : https://packagist.org/packages/sebk/small-orm-forms
* github : https://github.com/sebk69/small-orm-forms

### small-swoft-auth

This is a simple jwt authentication for swoft that use small-orm.

It come with an abstract class to protect your controllers with near no code and use sebk/swoft-voter to implement your custom rights management.

You can take it via :
* composer : https://packagist.org/packages/sebk/small-swoft-auth
* github : https://github.com/sebk69/small-swoft-auth

### sebk/swoft-voter

This is not really a small-om package but I mention it here because it is used by small-swoft-auth.

This package implement a voter system for Swoft, inspired by Symfony voters.

You can take it via :
* composer : https://packagist.org/packages/sebk/swoft-voter
* github : https://github.com/sebk69/swoft-voter
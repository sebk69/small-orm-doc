# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 11 : Useful commands

### Creating DAO and models from database

First useful command : helper to create DAO and models from database.

#### Symfony

```bash
$ bin/console sebk:small-orm:add-table
Connection [default] ?
Bundle [TestBundle] ?
Database table [all] ? customer
```

Just answer these three questions. By default, the bundle is the first configured bundle and if you don't specify a table, all database will be imported.

#### Swoft

```bash
$ bin/swoft sebk:small-orm:add-table --connection default --bundle TestBundle --table customer
```

Same as symfony bundle but the parameters are in command.

#### Result

This command will generate a DAO in the 'Dao' folder and an empty model in 'Model' folder  of the specified bundle by reverse engineering the db table. It's work only on MySql database for now.

The build method of DAO will be filled from database but defaults values and types are not managed.

### Add @method comments in model

Your models don't have any method : magic calls are used to manage getters and setters. So the IDE don't know your fields getters and setters.

It's a good idea to use @method to your class but it is a long work. This command do the job for you.

#### Symfony

```bash
$ bin/console sebk:small-orm:add-methods-bloc-comment
Connection [default] ?
Bundle [TestBundle] ?
Dao [all] ?
```

As for add-table, you'll have some questions to answer. You can specify DAO (faster) or not (easier) depends what you prefer.

#### Swoft

```bash
$ bin/swoft sebk:small-orm:generate:model-autocompletion --connection default --bundle TestBundle --dao Customer
```

Same as symfony bundle but the parameters are in command.

### Execute layers

We saw this command in chapter 8 : it will execute layers in current installation of your application.

No parameters are require : all necessary layers of all bundles will be executed.

#### Symfony

```bash
$ bin/console sebk:small-orm:layers-execute
```

#### Swoft

```bash
$ bin/swoft sebk:small-orm:layers-execute
```

### Generate CRUD

This command is a development helper that generate a CRUD for a specified DAO.

It is now implemented only for Swoft (see [chapter 12](chapter-12.md) for details).

#### Swoft

```bash
$ bin/swoft sebk:small-orm:generate:crud --bundle TestBundle --model Customer --template AuthToken --base-route person
```

### [legacy] Update DAO

This command has been developed before type management in small-orm DAO.

It's still working but override your build customizations in DAO then if you have typed of added defaults value, this customization will be lost.

So, use it only if you don't use field typing in small-orm.
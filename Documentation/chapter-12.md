# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 11 : Useful commands

### Creating DAO and models from database

First useful command : helper to create DAO and models from database.

#### Symfony

```bash
$ bin/console sebk:small-orm:generate:dao --connection default --bundle TestBundle --table customer
```

#### Swoft

```bash
$ bin/swoft sebk:small-orm:add-table --connection default --bundle TestBundle --table customer
```

#### Result

This command will generate a DAO in the 'Dao' folder and an empty model in 'Model' folder  of the specified bundle by reverse engineering the db table. It's work only on MySql database for now.

The build method of DAO will be filled from database but defaults values and types are not managed.

### Add @method comments in model

Your models don't have any method : magic calls are used to manage getters and setters. So the IDE don't know your fields getters and setters.

It's a good idea to use @method to your class but it is a long work. This command do the job for you.

#### Symfony

```bash
$ bin/console sebk:small-orm:generate:model-autocompletion --connection default --bundle TestBundle --dao Customer
```

You can specify DAO (faster) or not (easier) depends what you prefer. In last case, all models will be updated.

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

It is now implemented only for Swoft (see [chapter 12](chapter-13.md) for details).

#### Swoft

```bash
$ bin/swoft sebk:small-orm:generate:crud --bundle TestBundle --model Customer --template AuthToken --base-route person
```
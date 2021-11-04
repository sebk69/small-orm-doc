# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 8 : Database layers : an easy database deployment

### What are database layers ?

Database layers is a concept to help you manage your production installations.

A database layer is a group of sql scripts that must be executed for a version of your application.

A simple console command execute all scripts of all layers that haven't been executed on the current installation of you're app.

It is useful when you have many installations of your applications and to manage the development of a lot of versions at same time.

For example, the developer john is programming the customer table in the ticket 1253 and the developer paul is programming the table provider in the next ticket 1254.

During programming, we don't know who will merge to master branch the first.

Database layers are here to easily manage this situation :
* The developer john develop his model in branch ticket-1253 and create the layer ticket-1253
* The developer paul do the same in branch ticket-1254

Three possibility :
* The branch ticket-1253 is installed before ticket-1254
* The branch ticket-1254 is installed before ticket-1253
* The two branches are installed at the same time

These three possibilities are transparent and the two developers haven't to manage them. Just run the console command at each deployment.

To trace the layers executed on an installation, a table '_small_orm_layers' is created and all executed layers are stored into.

### Creating a database layer

The layers are located in : [your bundle folder] > 'Resources' > 'databaseLayers' > [name of your layer].

Usually, the layer name is your branch name.

It must contains : 
* a 'config.yml' file
* a sub folder 'scripts' containing all your sql scripts in text format

The simplest 'config.yml' must specify the connection to use for your script :
```yaml
connection: default
```

The scripts will be executed in alphanumeric order, then you are recommended to prefix your scripts by a numeric order.

### Execute layers for installation

In Symfony :
```bash
bin/console sebk:small-orm:layers-execute
```

In Swoft :
```bash
bin/swoft sebk:small-orm:layes-execute
```

### config.yml parameters

We saw 'connection' parameter previously but here are some non required parameters :

#### depends parameter

Sometimes your branch is based on another branch. For example paul develop a new field 'credit_card' in customer table. If the table doesn't exists, the script will fail. You can specify here that a layer must be executed before yours :
````yaml
connection: default
depends:
  - ticket-1253
````

You can specify as many of layers must be executed before yours as you want.

#### required-parameters

When you have more than one installations environments, some initializations in your application are specific for an environment. For example, inserting business configurations : you'll have two layers, one for business unit 1 and business unit 2.

You can specify here your condition to determine if your layer must be executed or not in install :
```yaml
connection: default
required-parameters:
  business-unit: "computer-sales"
```

The layer will be executed only if container parameter 'business-unit' have 'computer-sales' as value.

You can have any conditions as you want.
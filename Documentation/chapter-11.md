# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 11 : Backup model state

Usefull functionality is backup. It save state of model that you can compare later with current value.

For example, you can see if name of customer has changed :

```php
// Backup model state
$customer->backup();

// Change model
$customer->setName('Zebulon');

// Test if modified
if ($customer->getName() != $customer->getBackup()->name) {
    echo 'Customer name has changed !';
}
```

By default, this method create only backup on the model and not on dependencies. To backup also dependencies you can specify true as parameter :
```php
// Backup model deeply
$user->backup(true);

// Change group name
$user->getGroup()->setName("New group name");

// Test if modified
if($user->getGroup()->getName() != $user->getGroup()->getBackup()->name) {
    echo "The group name has been updated";
}

// Or test any modification on model since last backup
if($user->modifiedSinceBackup()) {
    echo "The user has been modified";
}
```

Notes :

* The backup is serialised on json_encode
* The metadata is also saved

It is also possible to rebuild old model :
```php
// Set name of user
$user->setName("Seb");

// Backup model
$user->backup();

// Change name
$user->setName("Opheli");

// Restore backup in $user2
$user2 = $dao->makeModelFromStdClass($user->getBackup());

// The two models names are ...
var_dump($user->getName()); // Output : Opheli
var_dump($user2->getName()); // Output : Seb
```
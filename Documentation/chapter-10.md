# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 10 : Forms an easy way to check input data and do partial update your models

### Introduction

small-orm-form is a set of classes to create data validation form.

You can use it to create simple forms to check input data from scratch, like login api.

It is also a powerful tool to create form automatically from a DAO and help you to do partial update of models.

### Installation

Require the package with composer:
```bash
composer require sebk/small-orm-forms
```

### Create simple form

The base of the form system is 'AbstractForm' abstract class. It's allowing you to easily create form classes to validate input data in api call.

Here is a simple example of login form class :
```php
<?php

use Sebk\SmallOrmForms\Form\AbstractForm;
use Sebk\SmallOrmForms\Form\Field;
use Sebk\SmallOrmForms\Type\StringType;

class LoginForm extends AbstractForm
{
    public function __construct(array $values)
    {
        $this->addField('account', 'User account', $values['account'] ?? null, StringType::TYPE_STRING, Field::MANDATORY);
        $this->addField('password', 'User password', $values['password'] ?? null, StringType::TYPE_STRING, Field::MANDATORY);
    }
}
```

Just define all fields of your form :
* First parameter is the field name, used by your main code to get or set fields attributes
* Second parameter is the field label, 'human' meaning of the field, used to generate error messages
* The third parameter is the value of the field
* The fourth parameter is the type of the string, used to check format of teh field (independently of the php type. ex : a '1' string value in a int type will generate a successful validation)
* The fifth parameter tell form if the field is mandatory or not, used to check the field is empty if the field is mandatory

Here are the possible types :
* StringType::TYPE_STRING
* BoolType::TYPE_BOOL
* IntType::TYPE_INT
* FloatType::TYPE_FLOAT
* DateTimeType::TYPE_DATE_TIME

To use it, just create object and call 'validate' method :
```php
$form = new LoginForm($this->loginForm, $data);
$messages = $form->validate();
if (count($messages) > 0) {
    foreach ($messages as $message) {
        echo $message->get();
    }
    return;
}

echo 'Success !';
```

### Use FormModel

To create model validation forms, you can just inject DAO object in a FormModel and set mandatory fields :
```php
use Sebk\SmallOrmForms\Form\FormModel;

$form = (new FormModel())
    ->buildFromDao(bean('sebk_small_orm_dao')->get('TestBundle', 'Customer'))
    ->setFieldMandatory('firstname')
    ->setFieldMandatory('lastname')
;
```

And then inject data for validation :
```php
// Fill form with body of request
$form->fillFromArray($request->getParsedBody());

// Get messages from validation
$messages = $form->validate();
if (count($messages) > 0) {
    // Validation failed
    return JsonResponse($messages)
        ->withStatus(Status::BAD_REQUEST)
    ;
}
```

Note that validation don't only generate technial checks on data : it call the model validator and merge form messages with validator messages.

You can also get a new model from form in order to persist your data :
```php
$model = $form->fillModel()->persist();

// return updated model
return JsonResponse($model);
```

As we have created and persist new model, we can inject a model to update it :
```php
// Get model to update
$model = bean('sebk_small_orm_dao')
    ->get('TestBundle', 'Customer')
    ->findOneBy($request->get('id'))
;

// Get data from body
$data = $request->getParsedBody();

// Create form and fill...
$form = (new FormModel())
    ->buildFromDao(bean('sebk_small_orm_dao')->get('TestBundle', 'Customer'))
    ->setFieldMandatory('firstname')
    ->setFieldMandatory('lastname')
    // with model
    ->fillFromModel($model)
    // and overwrite with body data
    ->fillFromArray($data)
;

// Get messages from validation
$messages = $form->validate();
if (count($messages) > 0) {
    // Validation failed
    return JsonResponse($messages)
        ->withStatus(Status::BAD_REQUEST)
    ;
}

// persist
$model = $form->fillModel()->persist();

// return updated model
return JsonResponse($model);
```

The last example will work with any partial update. For example the request can contains only :
```json
{
  "lastname":"skywalker"
}
```

Then this code will only update the lastname.

If your body is :
```json
{
  "lastname":""
}
```

The the code will respond a 400 http code with the body :
```json
[
  "The field lastname is mandatory"
]
```
# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 7 : Validators

### Purpose of validators

Before persist an object, it is a good idea to check technical and functional rules of the model to persist.

Another good idea, is to centralize you validation code : the persist process can be done by different parts of code.

The validator is a class to repond to that purpose. In addition, it give you some generics methods that can help you to implements some rules which are often used like check unicity of a code field and others.

### Validator class

A validator is a class that extends AbstractValidator abstract class. It must implement a 'validate' method returning a boolean of true value if validation is success.

It must be in the 'Validator' folder of your bundle and hase the same name as the model class.

Here is a simple validator that check the 'firstname' of customer cannot be empty :
```php
<?php

namespace app\Model\TestBundle\Validator;

use Sebk\SmallOrmCore\Validator\AbstractValidator;

class Customer extends AbstractValidator
{

    /**
     * Validate a customer model
     * @return bool
     */
    public function validate()
    {
        // Initialisations
        $result = true;
        $message = "";
        
        // The firstname cannot be empty
        if (!$this->testNonEmpty('firstname')) {
            $result = false;
            $message .= "The firstname cannot be empty\n";
        }
        
        // Result
        $this->message = $message;
        return $result;
    }

}
```

Note the message property : it will be useful to your controller to return the reason of why the validation failed to the user who submit the persist request.

The type of the message is up to you : choose the more appropriate format for your application. It can be string, array or application message class.

### Use validator in your persist process

You are free to implement the validator in your persist process. It will also depend of the model use : some technical models haven't to be validated, then will not have Validator class.

Here is an example of persist process for our customer :
```php
if (!$customer->getValidator()->validate()) {
    throw new \Exception($customer->getValidator()->getMessage());
}
$customer->persist();
```

### Helper methods

As we seen before, the 'AbstractValidator' implement some helper methods.

We have used the 'testNonEmpty' helper in our customer validation. This method take a field name in parameter and return false if the field is null or an empty string.

Here are other helper classes :
* 'testUnique' : take a field name in parameter and return false if the field is unique in database table
* 'testUniqueWithDeterminant' :
  * Input : 
    * determinant field
    * determinant value
    * field to test
  * Output : return false if the field to test is not unique in record having determinant value in determinant field
* 'testInteger'
  * test is field complient with int type in database

Note that 'testNonEmpty' and 'testInteger' are format validation and it is now recommended to use forms to do that (in small-orm-forms package).
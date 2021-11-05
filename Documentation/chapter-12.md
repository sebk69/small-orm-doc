# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 12 : Swoft CRUD generator

### Command

Sowft package come with a CRUD generator. It is not now available for Symfony bundle.

small-orm-forms and small-orm-auth packages are required.

The command will generate a new controller with these routes :
* GET 'baseRoute/model/{id}' : return the model with id
* POST 'baseRoute/model' : create a new model and persist
* POST 'baseRoute/model/{id}' : update the model with id
* DELETE 'baseRoute/model/{id}' : delete model with id

```bash
$ bin/swoft sebk:small-orm:generate:crud --bundle TestBundle --model Customer --template authToken --base-route person
```

### parameters

* bundle : the bundle of model for which the CRUD will be generated
* model : the model for which the CRUD will be generated
* template : optional, the template which will be used to generate CRUD
  * default : simple CRUD with no authentication
  * authToken : generate a CRUD for use with small-swoft-auth authentication
* base-route : optional, a prefix for route

### Example

For example we want a group of controllers concerning persons, and we want to generate a customer controller.

Just type this command :
```bash
$ bin/swoft sebk:small-orm:generate:crud --bundle TestBundle --model Customer --template authToken --base-route person
```

The generated controller will be placed in specified 'crudBasePath' config and will be named '[model name]Controller'.

Here is the generated code for our customer :
```php
<?php
namespace App\Http\Controller;

use App\Model\TestBundle\Model\Customer;

use Sebk\SmallSwoftAuth\Controller\TokenSecuredController;

use Sebk\SmallOrmCore\Dao\DaoEmptyException;
use Sebk\SmallOrmForms\Form\FormModel;
use Sebk\SmallOrmSwoft\Traits\Injection\DaoFactory;
use Sebk\SwoftVoter\VoterManager\VoterInterface;

use Swoft\Http\Message\Request;
use Swoft\Http\Server\Annotation\Mapping\Controller;
use Swoft\Http\Server\Annotation\Mapping\RequestMapping;
use Swoft\Http\Server\Annotation\Mapping\RequestMethod;
use Swoft\Http\Server\Annotation\Mapping\Middleware;
use Swoft\Http\Server\Annotation\Mapping\Middlewares;

use Swoft\Auth\Middleware\AuthMiddleware;

use Swoole\Http\Status;

/**
 * @Controller("person")
 * @Middlewares ({AuthMiddleware::class})
 * @Middleware (AuthMiddleware::class)
 */
class CustomerController extends TokenSecuredController
{
    use DaoFactory;

    /**
     * Create form for Customer
     * @return FormModel
     * @throws \Sebk\SmallOrmForms\Form\FieldException
     * @throws \Sebk\SmallOrmForms\Type\TypeNotFoundException
     */
    protected function createForm(): FormModel
    {
        return (new FormModel())
            ->buildFromDao($this->daoFactory->get('TestBundle', 'Customer'))
        ;
    }

    /**
     * @RequestMapping ("{id}", method={"GET"})
     * @param int $id
     * @return \Swoft\Http\Message\Response
     */
    public function getCustomer(int $id)
    {
        // Check user rights
        $this->denyAccessUnlessGranted(VoterInterface::ATTRIBUTE_READ, $this);

        // Load model
        try {
            /** @var Customer $model */
            $model = $this->daoFactory->get('TestBundle', 'Customer')->findOneBy(['id' => $id]);
        } catch (DaoEmptyException $e) {
            // Not found
            return JsonResponse('')
                ->withStatus(Status::NOT_FOUND)
            ;
        }

        return JsonResponse($model);
    }

    /**
     * @RequestMapping ("", method={"POST"})
     * @param Request $request
     * @return \Swoft\Http\Message\Response
     * @throws \Exception
     */
    public function createCustomer(Request $request)
    {
        // Check user rights
        $this->denyAccessUnlessGranted(VoterInterface::ATTRIBUTE_WRITE, $this);

        // Form validation
        $form = $this->createForm()
            ->fillFromArray($request->getParsedBody());
        $messages = $form->validate();
        if (count($messages) > 0) {
            // Validation failed
            return JsonResponse($messages)
                ->withStatus(Status::BAD_REQUEST);
        }

        // Persist
        $model = $form->fillModel()->persist();

        return JsonResponse($model);
    }

    /**
     * @RequestMapping ("{id}", method={"POST"})
     * @param int $id
     * @param Request $request
     * @return \Swoft\Http\Message\Response
     * @throws \Sebk\SmallOrmForms\Form\FieldException
     * @throws \Sebk\SmallOrmForms\Form\FieldNotFoundException
     * @throws \Sebk\SmallOrmForms\Type\TypeNotFoundException
     */
    public function patchCustomer(int $id, Request $request)
    {
        // Check user rights
        $this->denyAccessUnlessGranted(VoterInterface::ATTRIBUTE_UPDATE, $this);

        // Load model
        try {
            /** @var Customer $model */
            $model = $this->daoFactory->get('TestBundle', 'Customer')->findOneBy(['id' => $id]);
        } catch (DaoEmptyException $e) {
            // Not found
            return JsonResponse('')
                ->withStatus(Status::NOT_FOUND)
            ;
        }

        // Init data
        $data = $request->getParsedBody();
        if (isset($data['id'])) {
            unset($data['id']);
        }

        // Form validation
        $form = $this->createForm()
            ->fillFromModel($model)
            ->fillFromArray($data)
        ;
        $messages = $form->validate();
        $model = $form->fillModel();

        if (count($messages) > 0) {
            // Validation failed
            return JsonResponse($messages)
                ->withStatus(Status::BAD_REQUEST)
            ;
        }

        // Persist
        $model->persist();

        return JsonResponse($model);
    }

    /**
     * @RequestMapping ("{id}", method={"DELETE"})
     * @param int $id
     * @return \Swoft\Http\Message\Response
     */
    public function deleteCustomer(int $id)
    {
        // Check user rights
        $this->denyAccessUnlessGranted(VoterInterface::ATTRIBUTE_DELETE, $this);

        // Load model
        try {
            /** @var Customer $model */
            $model = $this->daoFactory->get('TestBundle', 'Customer')->findOneBy(['id' => $id]);
        } catch (DaoEmptyException $e) {
            // Not found
            return JsonResponse('')
                ->withStatus(Status::NOT_FOUND)
            ;
        }

        // Persist
        $model->delete();

        return JsonResponse("");
    }
}
```

You can now complete the code with your application specificities.
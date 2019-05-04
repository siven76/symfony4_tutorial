# Controllers

## Controller
- PHP function or method of an object
- purpose: read data from request and return a new response object
- can be any callable (function, method, or closure)
- usually reside inside controller classes (for convenience)
- best practice: 
	- all controllers should be inside the Controller namespace (folder)
	- should have the `Controller` suffix

### Routes
- URLs are mapped to controllers via Routing
	- route annotation above the controller
		- defines the URL of the controller (e.g. "/")
		- also define a name (e.g. "blog_index") which is useful to generate links

**Example**
```php
/**
 * @Route("/", name="blog_index")
 */
public function index(Request $request)
{
    return $this->render(
        'base.html.twig',
        ['message' => $this->greeting->greet($request->get('name'))]
    );
}
```

### Render
- a helper method to render a Twig template

### AbstractController
- all controllers are defined as services under the `services.yaml`
- nevertheless, controller classes extends the AbstractController
- AbstractController
	- one of the 2 possible base classes for a controller class
	- a convenient class which gives most of the often used features (e.g. the render method)
	- does not load all services available in the container, only the following
		- router
		- request_stack
		- http_kernel
		- serializer
		- session
		- security.authorization_checker
		- templating
		- twig
		- doctrine
		- form.factory
		- security.token_storage
		- security.csrf.token_manager
	- more efficient
	- to enforce usage of dependency injection

### Controller (base controller)
- have access to the full service container

#### Experiment 1: Fetching a custom service from the AbstractController
- to show that the AbstractController does not have access to all services defined in the container.

Assuming that the `app.greeting` service alias has been defined as a public service
```yaml
app.greeting:
    public: true
    alias: App\Service\Greeting
```

1. Add a line to the controller to use the Greeting service

```php
/**
 * @Route("/", name="blog_index")
 */
public function index(Request $request)
{
    // fetch the app.greeting service from the service container
    // this will generate an error when the controller extends the AbstractController
    $this->get('app.greeting');
    
    return $this->render(
        'base.html.twig',
        ['message' => $this->greeting->greet($request->get('name'))]
    );
}
```

2. Refresh the page (/?name=Siven)
```
Service "app.greeting" not found: the container inside "App\Controller\BlogController" is a smaller service locator that only knows about the "doctrine", "form.factory", "http_kernel", "parameter_bag", "request_stack", "router", "security.authorization_checker", "security.csrf.token_manager", "security.token_storage", "serializer", "session" and "twig" services.
```
- error: the `app.greeting` service is not available from the AbstractController

#### Experiment 2: Fetching a custom service from the Controller (base controller)
- not a good practice; not recommended

1. Change the `BlogController` class to extends the `Controller` class instead
```php
# src/Controller/BlogController.php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
#...

class BlogController extends Controller
{
	#...
}
```

2. Refresh the browser

```
Hello from service Siven
```

## Action Argument Resolver
- dependency injection also works for arguments of an action method
- the arguments must be type-hinted to be recognized
- this is done via the `[ArgumentResolver](https://github.com/symfony/symfony/blob/4.2/src/Symfony/Component/HttpKernel/Controller/ArgumentResolver.php)`
- the action argument resolver can be extended by creating and registering custom value resolvers
- e.g. when using the Request object as a controller argument


```php
// 
public function index(Request $request)
```

### Request parameters
- instead of using the Request object to get query parameters, placeholders can be used in the route definition
- the Request object is then not required
- the placeholder can be used as a variable (e.g. `{name}`)
- values can be passed directly in the URL without using query parameters
	- e.g. /Siven
	- the value 'Siven' will be passed as `name` to the action controller
	
```php
# src/Controller/BlogController.php

namespace App\Controller;

use App\Service\Greeting;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @var Greeting
     */
    private $greeting;

    /**
     * BlogController constructor.
     */
    public function __construct(Greeting $greeting)
    {
        $this->greeting = $greeting;
    }

    /**
     * @Route("/{name}", name="blog_index")
     */
    public function index($name)
    {
        return $this->render(
            'base.html.twig',
            ['message' => $this->greeting->greet($name)]
        );
    }
}
```


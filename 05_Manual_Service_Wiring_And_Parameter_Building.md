# Manual Service Wiring & Parameter Building

## Manual Service Wiring
- a few cases exist when an argument to a service cannot be autowired
	- examples:
		- configurable admin email in the constructor of a service
		- configurable text message for a greeting service

### Example: string argument in a constructor
- Add a string argument to the constructor of Greeting service
```php
# src/Service/Greeting.php

namespace App\Service;

use Psr\Log\LoggerInterface;

class Greeting
{
    /**
     * @var LoggerInterface
     */
    private $logger;
    /**
     * @var string
     */
    private $message;

    /**
     * Greeting constructor.
     */
    public function __construct(LoggerInterface $logger, string $message)
    {
        $this->logger = $logger;
        $this->message = $message;
    }

    public function greet(string $name): string
    {
        $this->logger->info("Greeted $name");

        return "{$this->message} $name";
    }
}
```
- Refresh the page (/?name=Siven)
	- Error
```
Cannot autowire service "App\Service\Greeting": argument "$message" of method "__construct()" is type-hinted "string", you should configure its value explicitly.
```
- Define the service manually
	- In `services.yaml`, add an entry for the service and pass a value for the argument `$message`
```yaml
App\Service\Greeting:
    arguments:
        $message: 'Hello from service'
```

**Note**
- by default `autowire` is already set to `true` in `services.yaml`
	- not necessary to specify the arguments for autowired services

- Refresh the page
	- Success: the page will now show `Hello from service Siven`

## Service Parameters
- the container also holds configuration, called `parameters`, in addition to service objects
- to create a parameter:
	- add it under the `parameters` key
	- reference it with the `%parameter_name%`
	- it can be referenced in any *other* configuration file
	- many are defined in the `config/services.yaml` file

**Example 1**
```yml
# config/services.yaml
parameters:
    admin_email: manager@example.com

services:
    # ...

    App\Updates\SiteUpdateManager:
        arguments:
            $adminEmail: '%admin_email%'

```

The parameter can be fetched in the service as follows:
```php
class SiteUpdateManager
{
    // ...

    private $adminEmail;

    public function __construct($adminEmail)
    {
        $this->adminEmail = $adminEmail;
    }
}

```

Parameters can also be fetched directly from the container:
```php
public function new()
{
    // ...

    // this shortcut ONLY works if you extend the base AbstractController
    $adminEmail = $this->getParameter('admin_email');

    // this is the equivalent code of the previous shortcut:
    // $adminEmail = $this->container->get('parameter_bag')->get('admin_email');
}

```

**Example 2**
- Define a parameter for the string
	- In the `services.yaml` file, add the following line under `parameters`
```yaml
parameters:
    locale: 'en'
    hello_message: 'Hello from service'
```
- Change the service definition to use the defined parameter

```yaml
App\Service\Greeting:
    arguments:
        $message: '%hello_message%'
```

- Refresh the page
	- Success: the page still works

## Binding Arguments by Name or Type
- the `bind` keyword can also be used to bind specific arguments by name or type

**Example**
```yaml
# config/services.yaml
services:
    _defaults:
        bind:
            # pass this value to any $adminEmail argument for any service
            # that's defined in this file (including controller arguments)
            $adminEmail: 'manager@example.com'

            # pass this service to any $requestLogger argument for any
            # service that's defined in this file
            $requestLogger: '@monolog.logger.request'

            # pass this service for any LoggerInterface type-hint for any
            # service that's defined in this file
            Psr\Log\LoggerInterface: '@monolog.logger.request'

            # optionally you can define both the name and type of the argument to match
            string $adminEmail: 'manager@example.com'
            Psr\Log\LoggerInterface $requestLogger: '@monolog.logger.request'

    # ...

```
- `bind` key under `_defaults`
	- specify the value of *any* argument for *any* service defined in *that* file
	- bind arguments 
		- by name (e.g. `$adminEmail`),
		- by type (e.g.. `Psr\Log\LoggerInterface`), or 
		- both (e.g. `Psr\Log\LoggerInterface $requestLogger`)

# Service Container

## Service
- a class that does something useful
	- examples:
		- a mailer object that sends emails
		- a cache object for caching data
		- Doctrine Entity Manager that fetch or persist data

## Service Container
- centralized place to control how services are constructed
- passing dependencies and parameters
- encourages best practices like Dependency Injection

## Symfony 4
** To see the list of services in Symfony 4 **
```bash
php bin/console debug:container
```

### services.yaml
- define services for the application
- for environment specific services use services_test.yaml for example

### Service directories
- all classes in src/ are available as services
- except classes under Entity, Migrations, Tests or Kernel.php
	- they do not do something useful

### Example: create a simple greeting service

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
     * Greeting constructor with LoggerInterface injected
     */
    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    /**
     * the function which does some work
     */
    public function greet(string $name): string
    {
        $this->logger->info("Greeted $name");

        return "Hello $name";
    }
}
```

### Example: using the Greeting service

```php
# src/Controller/BlogController.php

namespace App\Controller;

use App\Service\Greeting;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
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
     * @Route("/", name="blog_index")
     */
    public function index(Request $request)
    {
        return $this->render(
            'base.html.twig',
            ['message' => $this->greeting->greet($request->get('name'))]
        );
    }
}
```

The message variable is then used in the template file:

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}
            {{ message }}
        {% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

In the above example, the Service Container injected the LoggerInterface into the constructor of the Greeting service.

To know which service is being used by the LoggerInterface:

```bash
php bin/console debug:autowiring LoggerInterface
```

Output:
```
Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching LoggerInterface)

 Describes a logger instance.
 Psr\Log\LoggerInterface (monolog.logger)
```

To see the definition of the monolog.logger service:

```bash
php bin/console debug:container monolog.logger
```

Output:
```
Information for Service "monolog.logger"
========================================

 ---------------- -------------------------------------------------------------------
  Option           Value
 ---------------- -------------------------------------------------------------------
  Service ID       monolog.logger
  Class            Symfony\Bridge\Monolog\Logger
  Tags             -
  Calls            pushProcessor, useMicrosecondTimestamps, pushHandler, pushHandler
  Public           no
  Synthetic        no
  Lazy             no
  Shared           yes
  Abstract         no
  Autowired        no
  Autoconfigured   no
 ---------------- -------------------------------------------------------------------

```
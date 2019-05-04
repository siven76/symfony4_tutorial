# Autowiring & Auto-Configuring Services

## services.yaml
- autowiring is enabled by default for all services defined in this file

To check if a service is autowired:

```bash
php bin/console debug:container "App\Service\Greeting"
```

Output:
```
Information for Service "App\Service\Greeting"
==============================================

 ---------------- ----------------------
  Option           Value
 ---------------- ----------------------
  Service ID       App\Service\Greeting
  Class            App\Service\Greeting
  Tags             -
  Public           no
  Synthetic        no
  Lazy             no
  Shared           yes
  Abstract         no
  Autowired        yes
  Autoconfigured   yes
 ---------------- ----------------------

```

### Experiment 1: disable autowiring in services.yaml

- Disable autowiring
	- In `services.yaml`, set `autowire: false`
- Visit the URL /?name=Siven in the browser
	- Error: the Greeting service is not automatically injected in the BlogController
```
Too few arguments to function App\Controller\BlogController::__construct(), 0 passed in /symfony_tutorials/var/cache/dev/ContainerSvay1Xd/getBlogControllerService.php on line 13 and exactly 1 expected
```
- Inject the Greeting service manually to the BlogController
	- In `services.yaml`, add the following line under `services`:
```yaml
App\Controller\BlogController: ['@App\Service\Greeting']
```
- Refresh the page in the browser
	- Error: the Logger service is not automatically injected in the Greeting Service
```
Too few arguments to function App\Service\Greeting::__construct(), 0 passed in /symfony_tutorials/var/cache/dev/ContainerUuhx4IQ/getBlogControllerService.php on line 14 and exactly 1 expected
```
- Inject the Logger service manually to the Greeting service
	- In `services.yaml`, add the following line under `services`:
```yaml
App\Service\Greeting: ['@monolog.logger']
```
- Refresh the page in the browser
	- Success: there is no error and the page displays `Hello Siven`

## Auto-Configuration
- automatically registers your services as commands, event subscribers, etc
- useful when creating classes that implement certain Symfony core features

## Experiment 2: auto-configure

- Create a new security voter class
```php
# src/Security/ExampleVoter.php

namespace App\Security;

use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\VoterInterface;

class ExampleVoter implements VoterInterface
{
    public function vote(TokenInterface $token, $subject, array $attributes)
    {
        // TODO: Implement vote() method.
    }

}
```
- Check how the service looks inside the container
```bash
php bin/console debug:container "App\Security\ExampleVoter"
```

Output:
```
Information for Service "App\Security\ExampleVoter"
===================================================

 ---------------- ---------------------------
  Option           Value
 ---------------- ---------------------------
  Service ID       App\Security\ExampleVoter
  Class            App\Security\ExampleVoter
  Tags             security.voter
  Public           no
  Synthetic        no
  Lazy             no
  Shared           yes
  Abstract         no
  Autowired        yes
  Autoconfigured   yes
 ---------------- ---------------------------

```

- Note that the `tags` option has automatically been set for the ExampleVoter
- Now, disable the `autoconfigure` option in `services.yaml`, by setting its value to `false`
- Check the service again in the console using `php bin/console debug:container "App\Security\ExampleVoter"`
```
Information for Service "App\Security\ExampleVoter"
===================================================

 ---------------- ---------------------------
  Option           Value
 ---------------- ---------------------------
  Service ID       App\Security\ExampleVoter
  Class            App\Security\ExampleVoter
  Tags             -
  Public           no
  Synthetic        no
  Lazy             no
  Shared           yes
  Abstract         no
  Autowired        yes
  Autoconfigured   no
 ---------------- ---------------------------

``` 
- Note that the class does not have the security.voter tag anymore
# Public Services, Private Services, and Service Aliases

## Public Services
- can be accessed directly from the container at runtime
```php
// only public services can be accessed in this way
$doctrine = $container->get('doctrine');
```
- it is not a good practice to access services directly from the container
	- use dependency injection instead
		- those services do not need to be public
- services can be made public on a service-by-service basis
	- set the `public` option to `true`
```yaml
# config/services.yaml
services:
    # ...

    App\Service\Foo:
        public: false

```

## Private Services
- can be used with Dependency Injection
- all services are private by default (`services.yaml`)
	- optimises the container by removing unused services
	- gives better errors:
		- when referencing a non-existant service:
			- the error will be displayed on *any* page
			- even if the code is not for that page
- not recommended to set all services to public
- instead use service aliases to make a particular service public

## Aliasing
- in order to use shortcuts to access some services
- can also alias non-public services

**Example**
```yaml
# config/services.yaml
services:
    # ...
    App\Mail\PhpMailer:
        public: false

    app.mailer:
        alias: App\Mail\PhpMailer
        public: true

```
Then, the PhpMailer service can be accessed directly in the container (bad practice;
```php
$container->get('app.mailer'); // Would return a PhpMailer instance
```

**Shortcut to alias a service (only in yaml)**
```yaml
# config/services.yaml
services:
    # ...
    app.mailer: '@App\Mail\PhpMailer'

```

### Define a service alias
- In `services.yaml`, under `services` add the following
```yaml
app.greeting:
    public: true
    alias: App\Service\Greeting
```

**Notes:**
- above configuration makes the Greeting service a public service
- gives an alias `app.greeting` for the service `App\Service\Greeting`

## Anonymous Services
- **only supported by XML and YAML configuration formats**
- to prevent a service from being used as Dependency Injection by other services
- like regular services but without an ID
- created where they are used

**Example: how to inject an anymous service into another service**
```yaml
# config/services.yaml
services:
    App\Foo:
        arguments:
            - !service
                class: App\AnonymousBar

```

**Example: using an anonymous service as a factory**
```yaml
# config/services.yaml
services:
    App\Foo:
        factory: [ !service { class: App\FooFactory }, 'constructFoo' ]

```

## Deprecating Services
- because it is outdated or not maintained anymore
- a deprecation warning will be triggered when used
- must have a least one occurrence of the `%service_id%` placeholder in the message
	- will be replaced by the service's id
- the deprecation message is optional
	- recommended to define a custom message
		- default one is too generic
	- informs when it was deprecated, until when it will be maintained and alternative services to use (if any)
**Example: to deprecate a service**
```yaml
# config/services.yaml
App\Service\OldService:
    deprecated: The "%service_id%" service is deprecated since 2.8 and will be removed in 3.0.

```
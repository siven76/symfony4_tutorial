# Public Services, Private Services, and Service Aliases

- all services are private by default (`services.yaml`)
	- optimises the container by removing unused services
- not recommended to set all services to public
- instead use service aliases to make a particular service public

## Define a service alias
- In `services.yaml`, under `services` add the following
```yaml
app.greeting:
	public: true
	alias: App\Service\Greeting
```

Notes:
- above configuration makes the Greeting service a public service
- gives a alias `app.greeting` for the service class `App\Service\Greeting`
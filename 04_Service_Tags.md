# Service Tags
- a way to tell Symfony or other third-party bundles that a service should be registered in some special way

**Example: registering a Twig extension**
```yaml
# config/services.yaml
services:
    App\Twig\AppExtension:
        public: false
        tags: ['twig.extension']

```
- services tagged with `twig.extension` tag are collected during initialization of TwigBundle
- those services are then added to Twig as extensions
- list of [built-in Symfony service tags](https://symfony.com/doc/current/reference/dic_tags.html)
- each tag has a different effect on the service
- many tags require additional arguments
- when auto-configure option is set to true in `services.yaml`
	- service tags are automatically added for the command

Experiment 3: auto-configure the service tags for a new console command

- Create a new console command class

```php
# src/Command/HelloCommand.php

namespace App\Command;

use App\Service\Greeting;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class HelloCommand extends Command
{
    /**
     * @var Greeting
     */
    private $greeting;

    /**
     * HelloCommand constructor.
     */
    public function __construct(Greeting $greeting)
    {
        $this->greeting = $greeting;

        parent::__construct();
    }

    protected function configure()
    {
        $this->setName('app:say-hello')
            ->setDescription('Says hello to the user')
            ->addArgument('name', InputArgument::REQUIRED);
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln([
            'Hello from the app',
            '==================',
            ''
        ]);
        $output->writeln($this->greeting->greet($name));
    }
}
```

- Check the service info in the container
```bash
php bin/console debug:container "App\Command\HelloCommand"
```

Output:
```
Information for Service "App\Command\HelloCommand"
==================================================

 ---------------- --------------------------
  Option           Value
 ---------------- --------------------------
  Service ID       App\Command\HelloCommand
  Class            App\Command\HelloCommand
  Tags             console.command
  Public           no
  Synthetic        no
  Lazy             no
  Shared           yes
  Abstract         no
  Autowired        yes
  Autoconfigured   yes
 ---------------- --------------------------

```

- Note the `tags` for the command has automatically been set to `console.command` as the HelloCommand extends the Command class

- Try the command

```bash
php bin/console app:say-hello Siven
```

Output:
```
Hello from the app
==================

Hello Siven

```
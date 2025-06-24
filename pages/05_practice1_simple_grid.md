---
layout: center
hideInToc: true
---

# Practice 1: let's generate a simple grid view

<img src="/basic_driver_grid.png">

---
hideInToc: true
---

# What do we need to generate an index grid?

* **routing & operations** (Sylius Resource attributes)
* a data **model** (can be a Sylius Resource)
* a **data provider** (Doctrine, API repo, etc)
* a **Sylius Grid** definition (structure: fields, filters, sorting, pagination and actions)

---

## 1- Create a Resource

```php {all|3-5|all}
#[AsResource(
    templatesDir: '@SyliusAdminUi/crud',
    operations: [
        new Index(grid: DriverGrid::class),
    ],
)]
final readonly class DriverResource implements ResourceInterface
{
    public function __construct(
        public int $number,
        public string $firstName,
        public string $lastName,
        public string $countryCode,
        public string $teamName,
        public string|null $image = null,
    ) {
    }

    public function getId(): int
    {
        return $this->number;
    }
}
```

---

## 2- Create a custom Grid Data Provider

<!-- empty provider
--> 

```php {all|10|6,10|12|12,5|14-15}
namespace App\Grid;

use Pagerfanta\Adapter\FixedAdapter;
use Pagerfanta\Pagerfanta;
use Pagerfanta\PagerfantaInterface;
use Sylius\Component\Grid\Data\DataProviderInterface;
use Sylius\Component\Grid\Definition\Grid;
use Sylius\Component\Grid\Parameters;

final readonly class DriverGridProvider implements DataProviderInterface
{
    public function getData(Grid $grid, Parameters $parameters): PagerFantaInterface
    {
        // start with an empty paginator
        return new Pagerfanta(new FixedAdapter(0, []));
    }
}
```

---

```php {all|7-8|11-17}
// ...

final readonly class DriverGridProvider implements DataProviderInterface
{
    public function getData(Grid $grid, Parameters $parameters): PagerFantaInterface
    {
        // start with a fixed data paginator
        return new Pagerfanta(new FixedAdapter(4, $this->getDrivers()));
    }

    private function getDrivers(): iterable
    {
        yield new DriverResource(number: 1, firstName: 'Max', lastName: 'Verstappen', countryCode: 'NED', teamName: 'Red Bull Racing', image: 'https://www.formula1.com/content/dam/fom-website/drivers/M/MAXVER01_Max_Verstappen/maxver01.png.transform/1col/image.png');
        yield new DriverResource(number: 2, firstName: 'Logan', lastName: 'Sargeant', countryCode: 'USA', teamName: 'Williams', image: 'https://www.formula1.com/content/dam/fom-website/drivers/L/LOGSAR01_Logan_Sargeant/logsar01.png.transform/1col/image.png');
        yield new DriverResource(number: 4, firstName: 'Lando', lastName: 'Norris', countryCode: 'GBR', teamName: 'McLaren', image: 'https://www.formula1.com/content/dam/fom-website/drivers/L/LANNOR01_Lando_Norris/lannor01.png.transform/1col/image.png');
        yield new DriverResource(number: 44, firstName: 'Lewis', lastName: 'Hamilton', countryCode: 'GBR', teamName: 'Mercedes', image: 'https://www.formula1.com/content/dam/fom-website/drivers/L/LEWHAM01_Lewis_Hamilton/lewham01.png.transform/1col/image.png');
    }
}
```


---
layout: image
image: '/fixed_data.png'
backgroundSize: contain
---

---
layout: center
class: bg-black text-white
---

Now, let's use real data from the API!

<img src="/starting_grid.gif">


---
transition: fade
---

```php {all|14,9|17-20}
namespace App\Grid\Provider;

use Pagerfanta\Adapter\ArrayAdapter;
use Pagerfanta\Pagerfanta;
use Pagerfanta\PagerfantaInterface;
use Sylius\Component\Grid\Data\DataProviderInterface;
use Sylius\Component\Grid\Definition\Grid;
use Sylius\Component\Grid\Parameters;
use Symfony\Contracts\HttpClient\HttpClientInterface;

final readonly class DriverGridProvider implements DataProviderInterface
{
    public function __construct(
        private HttpClientInterface $openF1Client,
    ) {}

    public function getData(Grid $grid, Parameters $parameters): PagerFantaInterface
    {
        return new Pagerfanta(new ArrayAdapter(iterator_to_array($this->getDrivers())));
    }

    private function getDrivers(): iterable
    {
        // ...
    }
}
```

---
transition: fade
---

```php {19-22}
namespace App\Grid\Provider;

use Pagerfanta\Adapter\ArrayAdapter;
use Pagerfanta\Pagerfanta;
use Pagerfanta\PagerfantaInterface;
use Sylius\Component\Grid\Data\DataProviderInterface;
use Sylius\Component\Grid\Definition\Grid;
use Sylius\Component\Grid\Parameters;
use Symfony\Contracts\HttpClient\HttpClientInterface;

final readonly class DriverGridProvider implements DataProviderInterface
{
    public function __construct(
        private HttpClientInterface $openF1Client
    ) {}

    // ...

    private function getDrivers(): iterable
    {
        // ...
    }
}
```


---

```php{all|10|13-21|all}

use Sylius\Component\Grid\Data\DataProviderInterface;
use Symfony\Contracts\HttpClient\HttpClientInterface;

final readonly class DriverGridProvider implements DataProviderInterface
{
    // ...
    
    private function getDrivers(): iterable
    {
        $responseData = $this->openF1Client->request('GET', '/v1/drivers?session_key=9158')->toArray();

        foreach ($responseData as $row) {
            yield new DriverResource(
                number: $row['driver_number'],
                firstName: $row['first_name'],
                lastName: $row['last_name'],
                countryCode: $row['country_code'],
                teamName: $row['team_name'],
                image: $row['headshot_url'],
            );
        }
    }
}
```

---
layout: image
image: '/grid_with_api.png'
backgroundSize: contain
---

---
layout: center
---

𝄜 Improved `make:grid` console command

---

## 3- Create a Grid

<v-clicks>

💡 <em>Now you can also create a grid based on a non-Doctrine entity, including... Sylius Resources !</em>

```shell
symfony console make:grid 'App\Resource\DriverResource'
```

<img src="/new_make_grid.png" class="w-150"/>

</v-clicks>

---


## 3- Create a Grid

```php
final class DriverResourceGrid extends AbstractGrid implements ResourceAwareGridInterface
{
    public function __construct() {  // TODO inject services if required  }
    
    public static function getName(): string
    {
        return 'app_driver_resource';
    }
    
    public function getResourceClass(): string
    {
        return DriverResource::class;
    }
    
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->addField(StringField::create('firstName')->setLabel('FirstName')->setSortable(true))
           // ... all fields are added except "id"
            ->addActionGroup(MainActionGroup::create(CreateAction::create()))
            ->addActionGroup(BulkActionGroup::create(DeleteAction::create()))
            ->addActionGroup(ItemActionGroup::create(
                    // ShowAction::create(),
                    UpdateAction::create(),
                    DeleteAction::create()))
        ;
    }
}

```

---

## 4- Add provider to grid

```php {all|6|all}
final class DriverGrid extends AbstractGrid implements ResourceAwareGridInterface
{
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->setProvider(DriverGridProvider::class)
            ->addField(
                StringField::create('firstName')
                ->setLabel('FirstName')
                ->setSortable(true)
                )
            // ...
        ;
    }
}
```


---
layout: image
image: '/grid_with_api.png'
backgroundSize: contain
---


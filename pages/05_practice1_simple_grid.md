---
layout: center
hideInToc: true
---

# Practice 1: let's generate a simple grid view

<img src="/basic_driver_grid.png">

<!--
*Estelle*

First objective
-->

---
hideInToc: true
---

# What do we need to generate an index grid?

* **routing & operations** (Sylius Resource attributes)
* a data **model** (can be a Sylius Resource)
* a **data provider** (Doctrine, API repo, etc)
* a **Sylius Grid** definition (structure: fields, filters, sorting, pagination and actions)

<!-- 
*Estelle*
-->

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

<!-- 
*Estelle*
-->

---
layout: center
---

𝄜 Improved `make:grid` console command

<!-- 
*Estelle*
-->

---

## 2- Create a Grid

<v-clicks>

💡 <em>Now you can also create a grid based on a non-Doctrine entity, including... Sylius Resources !</em>

```shell
symfony console make:grid 'App\Resource\DriverResource'
```

<img src="/new_make_grid.png" class="w-150"/>

</v-clicks>

<!--
*Estelle*

The make:grid command, which already existed before, is great to generate grids based on Doctrine entities, however in my case here, I don't actually have a Doctrine entity. So we improved this command to be able to run it with any PHP entity or DTO, including... Sylius Resources themselves ! So now, I'm able to run this command with my resource and it will generate the grid configuration for me.

Any PHP object can be used to generate a grid based on its properties.
-->

---

<!-- 
*Estelle*
-->


## 2- Create a Grid

```php {all|9-26|all}
#[AsGrid(
    resourceClass: DriverResource::class,
    name: 'app_driver_resource',
)]
final class DriverResourceGrid extends AbstractGrid
{
    public function __construct() {  // TODO inject services if required  }
    
    public function __invoke(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->addField(StringField::create('firstName')->setLabel('FirstName')->setSortable(true))
           // ... all fields are added except "id"
            ->addActionGroup(MainActionGroup::create(
                CreateAction::create()
            ))
            ->addActionGroup(BulkActionGroup::create(
                DeleteAction::create()
            ))
            ->addActionGroup(ItemActionGroup::create(
                // ShowAction::create(),
                UpdateAction::create(),
                DeleteAction::create())
            )
        ;
    }
}

```

<!--
*Estelle*

Generated grid
* grid name
* associated resource
* invokable to build the grid (SRP)
-->

---
layout: image
image: "/doctrine_error.png"
---

<!--
*Loïc*

Oh, la, la Estelle ! what have you done?!

Ok, the error message is clear, sorry. Doctrine does not manage our DriverResource class, cause it's not a Doctrine Entity.

Thus, it needs a custom grid data provider.
-->

---

## 3- Add provider to grid

```php {all|4|all}
#[AsGrid(
    resourceClass: DriverResource::class,
    name: 'app_driver_resource',
    provider: DriverGridProvider::class,
)]
```

<!--
*Loïc*

To use a custom provider on a grid, we declare it on the AsGrid attribute using the Fully classified class name.
-->

---

## 4- Create a custom Grid Data Provider
👶 3 Baby steps

<v-clicks>

1. Without any data
2. With hardcoded data
3. With data from the F1 API

</v-clicks>

<!--
*Loïc*

To create this custom grid provider and in order to make thinks simpler for you to understand, I will use three baby steps.

* First, we'll create it without any data.
* After, we'll use hardcoded data.
* And finally we'll use data from the F1 API
-->

---

## 1. Without any data

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

<!--
*Loïc*

Now, let's see how to write your own data provider.
It needs to implement the DataProviderInterface.

You can return everything you want but the default templates will handle a pagerfanta object.
To do so, we can start slowly with an empty paginator.
-->

---

<img src="/empty_data.png">

<!--
*Loïc*

It works, everything is well configured.
-->

---

## 2. Use hardcoded data

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

<!--
*Loïc*

So now, in the second step, we can set the driver list with hardcoded data.

Here we use a FixedAdapter.

And we instanciate four  Driver Resources directly in our data provider.
We specify drivers' data in the DriverResource constructor.
-->

---
layout: image
image: '/fixed_data.png'
backgroundSize: contain
---

<!--
*Loïc*

Fantastic, 4 drivers appear in a beautiful data table.
-->

---
layout: center
class: bg-black text-white
---

3. Now, let's use real data from the API!

<img src="/starting_grid.gif">

<!-- 
*Loïc*

And finally, let's use real data from the API.

-->

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

<!--
*Loïc*

We have configured Symfony HTTP client to retrieve data from the API.

We use the ArrayAdapter from Pagerfanta
-->

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

<!-- 
*Loïc*
-->

---

```php{all|10-21}

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

<!--
*Loïc*

and we use the response data to instantiate the drivers.
-->

---
layout: image
image: '/grid_with_api.png'
backgroundSize: contain
---

<!--
*Loïc*

Good job !

All the twenty drivers appear in the list.

It's pretty easy ! Right?
-->

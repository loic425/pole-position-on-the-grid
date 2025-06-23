---
layout: center
---

## Practice 3 : 🔎 Filter for specific drivers

using the brand new #[AsFilter] attribute

---
transition: fade
---


## #[AsFilter]
```php {all|8,3|9,6|10|12-18,5|all}
namespace App\Grid\Filter;

use Sylius\Component\Grid\Attribute\AsFilter;
use Sylius\Component\Grid\Data\DataSourceInterface;
use Sylius\Component\Grid\Filtering\FilterInterface;
use Symfony\Component\Form\Extension\Core\Type\CountryType;

#[AsFilter(
    formType: CountryType::class, // Symfony Form Type to use
    template: '@SyliusBootstrapAdminUi/shared/grid/filter/select.html.twig', // The Twig template
)]
final class CountryFilter implements FilterInterface
{
    public function apply(DataSourceInterface $dataSource, string $name, $data, array $options): void
    {
        // We handle the filtering part in the DriverGridProvider
    }
}
```

---

## Insert the filter into our Grid

```php {all|7,14|8-12|all}
#[AsGrid]
final class DriverGrid extends AbstractGrid
{
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->addFilter(
                Filter::create('country', CountryFilter::class)
                    ->setFormOptions([
                        'alpha3' => true,
                        'autocomplete' => true,
                    ])
                    ->setLabel('app.ui.country')
            )
            // ...
        ;
    }
}

```

---
transition: fade
---

## Actual filtering logic inside the provider

```php {all|9-11|14|26|all}
final readonly class DriverApiGridProvider implements DataProviderInterface
{
    // ...

    private function getDrivers(array $criteria): iterable
    {
        $query = ['session_key' => 9158];
    
        if (!empty($criteria['country'] ?? null)) {
            $query['country_code'] = $criteria['country'];
        }
    
        $responseData = $this->openF1Client
            ->request(method: 'GET', url: '/v1/drivers', options: ['query' => $query])
            ->toArray()
        ;

        foreach ($responseData as $row) {
            yield new DriverResource(
                number: $row['driver_number'],
                // ...
            );
        }
    }
}
```

---
transition: fade
layout: image
image: '/country_filter.png'
backgroundSize: contain
---

---
layout: image
image: '/country_filter_results.png'
backgroundSize: contain
---

---
layout: center
---

Add a link to another grid with filtered data

---
layout: image
image: '/team_radio.png'
backgroundSize: contain
---

---

```php
namespace App\Grid;

use Sylius\Bundle\GridBundle\Builder\Filter\StringFilter;
use Sylius\Bundle\GridBundle\Builder\GridBuilderInterface;
use Sylius\Bundle\GridBundle\Grid\AbstractGrid;
use Sylius\Component\Grid\Attribute\AsGrid;

#[AsGrid]
final class TeamRadioGrid extends AbstractGrid
{
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            // ...
            ->addFilter(
                StringFilter::create(name: 'driver_number', type: 'equal')
                    ->setLabel('app.ui.driver_number')
            )
            // ...
        ;
    }
}
```

---

```php {all|11,2|12,1|13-24}
use Sylius\Bundle\GridBundle\Builder\Action\Action;
use Sylius\Bundle\GridBundle\Builder\ActionGroup\ItemActionGroup;

#[AsGrid]
final class DriverGrid extends AbstractGrid
{
    public function buildGrid(GridBuilderInterface $gridBuilder): void
    {
        $gridBuilder
            ->addActionGroup(
                ItemActionGroup::create(
                    Action::create('team_radios', 'show')
                        ->setOptions([
                            'link' => [
                                'route' => 'app_admin_team_radio_index',
                                'parameters' => [
                                    'criteria' => [
                                        'driver_number' => [
                                            'value' => 'resource.number', // driverResource->number
                                        ],
                                    ],
                                ],
                            ],
                        ])
                        ->setLabel('app.ui.show_team_radios')
                        ->setIcon('tabler:radio'),
                )
            )
        ;
    }
}
```

---
transition: fade
layout: image
image: '/driver_with_linked_action.png'
backgroundSize: contain
---

---
layout: image
image: '/filtered_team_radio.png'
backgroundSize: contain
---


---

## [#AsFilter]


<v-clicks>

* **formType** argument instead of getFormType()

* **template** argument instead of defining in config/packages/sylius_grid.php

</v-clicks>


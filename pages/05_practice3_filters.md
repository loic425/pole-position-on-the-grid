---
layout: center
---

## Practice 3 : 🔎 Filter for specific drivers

using the brand new #[AsFilter] attribute

<!--
*Estelle*
-->

---
layout: two-cols
---

Filter types:

<v-clicks>

- String
- Boolean
- Date
- Entity
- Money
- Exists
- Select
- <span v-mark="{ at: 5, color: 'red', type: 'circle' }">Custom</span>
 
</v-clicks>

::right::

**String filter example**
![img.png](/string_filter.png)

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

<!--
*Estelle*

Custom grid data provider, the filtering logic will be applied directly inside the provider "request"

* Filter definition
* Corresponding  Symfony Form Type
* Template to use
-->

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

<!--
*Estelle*

Country filter comes from Symfony
-->

---
transition: fade
---

## Actual filtering logic inside the provider

```php {all|9-11|14|19-22|all}
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

<!-- 
*Estelle*
-->

---
transition: fade
layout: image
image: '/country_filter.png'
backgroundSize: contain
---

<!-- 
*Estelle*
-->

---
layout: image
image: '/country_filter_results.png'
backgroundSize: contain
---

<!-- 
*Estelle*
-->

---

## [#AsFilter]


<v-clicks>

* **formType** argument instead of getFormType()

* **template** argument instead of defining in config/packages/sylius_grid.php

* **type** argument for the name of your custom filter type (FQCN by default)
  * default ones are `string`, `boolean`, `money`, `date`...

</v-clicks>

<!-- 

**Template**

Avant, il fallait définir le template du filter dans la configuration du bundle.

**Type**

La confusion possible entre le type et le formType disparait.
Le type étant défini automatiquement avec le FQCN, on n'a plus à s'en préoccuper.

-->

<!-- 
*Estelle*
-->

---
layout: center
---

## Connect Grids together
Add a link to another grid with filtered data

<!-- 
*Estelle*
-->

---
layout: image
image: '/team_radio.png'
backgroundSize: contain
---

<!-- 
*Estelle*
-->

---

```php {all|16-17|all}
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

<!-- 
*Estelle*
-->

---

```php {all|12,1|13-24}
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

<!-- 
*Estelle*
-->

---
transition: fade
---

<div class="relative w-full">
  <img src="/driver_with_linked_action.png" class="w-full" />

  <div
    class="absolute border-4 border-red-500 rounded-full w-16 h-32"
    style="top: 35%; left: 91%;"
    v-click
  ></div>
</div>

<!-- 
*Estelle*
-->

---
layout: image
image: '/filtered_team_radio.png'
backgroundSize: contain
---

<!--
*Estelle*

This feature is actually very useful and we've taken full advantage of it on our client projects to create many different prefiltered grids and links between grids.
-->

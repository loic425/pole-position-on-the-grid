---
layout: center
hideInToc: true
---


# Practice 2: 𝄜 The brand new #[AsGrid] attribute

<!--
*Estelle*
-->

---
layout: center
---

## Resource class & provider arguments

```php {all|5|6|10-14|all}
use App\Entity\Meeting;
use Sylius\Bundle\GridBundle\Grid\AbstractGrid;

#[AsGrid(
    provider: MeetingGridProvider::class,
    resourceClass: Meeting::class,   // any PHP object, including a Sylius resource
)]
final class MeetingGrid extends AbstractGrid
{
    public function __invoke(
        GridBuilderInterface $gridBuilder,
    ): void 
        // ...
    }
}
```

<!--
*Estelle*

Quand on utilise un grid provider custom, on n'a pas besoin de spécifier la resource class qui est utilisée par les drivers Doctrine.
-->

---

## build method and name arguments

```php {all|4,10|5|all}
use Sylius\Bundle\GridBundle\Grid\AbstractGrid;

#[AsGrid(
    buildMethod: 'customBuildMethod', 
    name: 'meeting', // optional - FQCN by default
    // ...
)]
final class MeetingGrid extends AbstractGrid
{
    public function customBuildMethod(
        GridBuilderInterface $gridBuilder,
    ): void 
        // ...
    }
}
```

<!--
*Estelle*

* __invoke => - SOLID single responsibility / separation of concerns 
              - consistency with Symfony DX for services to be autowirable callables
              - no need to implement an interface anymore
* flexibility : you can still use buildGrid (it works behind the scenes without interface)
* custom build method => - Multiple grids def in one class
                         - Reusable logic (traits/base)
                         - Decorators/extensions
#[AsGrid('app_admin_user')]
#[AsGrid('app_admin_customer', buildMethod: 'buildCustomerGrid')]
final class UserGrids
{
    public function __invoke(GridBuilderInterface $grid): void
    {
        // User grid definition
    }

    public function buildCustomerGrid(GridBuilderInterface $grid): void
    {
        // Customer grid definition
    }
}
-->


---
layout: center
---

## [#AsGrid]


<v-clicks>

* **__invoke()** instead of buildGrid() - but buildGrid still supported by default if no __invoke()

* **buildMethod** argument to declare custom build method

* **provider** argument instead of ->setProvider()

* **resourceClass** argument instead of getResourceClass()

* **name** argument (FQCN by default) instead of getName()

</v-clicks>

<!--
*Estelle*

* __invoke => - SOLID single responsibility / separation of concerns 
              - consistency with Symfony DX for services to be autowirable callables
              - no need to implement an interface anymore
* flexibility : you can still use buildGrid (it works behind the scenes without interface)
* custom build method => - Multiple grids def in one class
                         - Reusable logic (traits/base)
                         - Decorators/extensions
#[AsGrid('app_admin_user')]
#[AsGrid('app_admin_customer', buildMethod: 'buildCustomerGrid')]
final class UserGrids
{
    public function __invoke(GridBuilderInterface $grid): void
    {
        // User grid definition
    }

    public function buildCustomerGrid(GridBuilderInterface $grid): void
    {
        // Customer grid definition
    }
}
-->

---
layout: image
image: '/grid_start.jpg'
---


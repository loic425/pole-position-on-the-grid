---
layout: center
image: /grid_start.jpg
backgroundSize: contain
---

# Quick practice before it's your time to shine!
Q1: Find the odd one out.

<v-clicks>

* ```#[AsFilter] ```
* ```#[AsGrid]```
* <span v-mark="{ at: 5, color: 'red', type: 'circle' }">```#[AsProvider]```</span>
* ```#[AsResource]```


</v-clicks>

<!--
*Estelle*
-->

---

Q2: Which team did Max Verstappen race for when he won his first Formula One Grand Prix?

<v-clicks>

* Ferrari
* Mercedes
* <span v-mark="{ at: 5, color: 'red', type: 'circle' }">Red Bull Racing</span>
* Scuderia Toro Rosso


</v-clicks>

<!--
*Loïc*
-->

---

Q3: The OpenF1 API does not include a **/teams** endpoint, but you would like to create a grid listing all teams.
You've created this simple model ... : 

```php
final readonly class Team
{
    public function __construct(
        public string $id,
        public string $name,
        public string|null $color = null,
    ) {
    }
}
```

<!--
*Estelle*
-->

---

Q3: What command could you run to generate the missing grid?

<v-clicks>

* ```symfony console create:grid 'App\Model\Team'```
* ```symfony console make:resource 'App\Model\Team'```
* <span v-mark="{ at: 5, color: 'red', type: 'circle' }">```symfony console make:grid 'App\Model\Team'```</span>
* ```symfony console make:grid 'App\Resource\TeamResource'```

</v-clicks>

<!--
*Estelle*

    Although technically, we still need to add a TeamResource to generate routing.
-->


---
layout: center  
hideInToc: true
---

# symfony console make:grid 'App\Model\Team'

<img src="/teams_grid.png">

<!--
*Loïc*
-->

---

Q4: How can you create a Team filter to use it on the F1 drivers' grid?

We've added the following filter on the grid configuration:

```php
  ->addFilter(Filter::create('teamName', TeamFilter::class)) 
```

Now, what should our filter look like?

<v-clicks>

* ```php
  #[AsFilter(formType: TeamFilterType::class, template: '@SyliusBootstrapAdminUi/shared/grid/filter/select.html.twig',)]
  final class TeamFilter implements FilterInterface


* ```php
  #[AsGridFilter(formType: TeamFilterType::class, template: '@SyliusBootstrapAdminUi/shared/grid/filter/select.html.twig',)]
  final class TeamFilter implements FilterInterface


* ```php
  #[Filter(formType: TeamFilterType::class, template: '@SyliusBootstrapAdminUi/shared/grid/filter/select.html.twig',)]
  final class TeamFilter implements FilterInterface


</v-clicks>


<!--
*Loïc*

We have our driver's grid.

And we've added this following filter on the grid configuration/

Now, what should our filter definition look like?

Do we need to use the `AsFilter` attribute?
AsGridFilter attribute, or, Filter attribute.
-->

---

Q5: During 1st World War, whose fighter plane used the horse 🐴 symbol that became Ferrari's logo?

<v-clicks>

* Alfredo Ferrari
* Tazio Nuvolari
* Enzo Ferrari
* <span v-mark="{ at: 5, color: 'red', type: 'circle' }">Francesco Baracca</span>


</v-clicks>

<!--
*Estelle*
-->



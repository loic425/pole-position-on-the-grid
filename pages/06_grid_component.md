---
layout: center
---

## Push, Push, Push!

<img src="/push_lewis.gif" class="w-120">

<!--
*Loïc*
-->

---
layout: center
name: mystery-slide
title: Mystery surprise 👀
---

# 🧟 New Grid Live Component

it's alive...

<!--
*Loïc*
-->

---
layout: image
image: /pit_stop.gif
---

<!--
*Loïc*
-->

---

## Twig hooks overview
index operation

```mermaid
flowchart LR
    Template(Index template) --> Hook{Hook 'index'}

    Hook --> Sidebar([Sidebar])
    Hook --> Navbar([Navbar])
    Hook --> Content([Content])
    
    Content --> HookContent{Hook 'content'}

    HookContent --> Flashes([Flashes])
    HookContent --> Header([Header])
    HookContent --> Grid([Grid])
```

<!--
*Loïc*

How can we make our grid, a LIVE grid ?

First, we need to look at our Twig template definition.

In Sylius, we have a Twig template tree / tree of blocks which we can customize.

We need to replace the default blocks to use our future grid component 

Main template : index template contains blocks
Twig Hooks definition to add more blocks inside the index template

BootstrapAdminUI configures these blocks by default
By default, our grid is just a template which we can override with our new grid component

Deïïïta table component

TODO: CIRCLE THE GRID BLOCK !!!!!!!
-->

---
transition: fade
---

## Twig hooks overview
index operation

<img src="/twig_hooks_profiler.png">

<!--
*Loïc*

Driver grid
in the Symfony Web Profiler page
Twig Hooks list
Grid Hook contains the data_table hook
-->

---

## Twig hooks updated
index operation

<img src="/twig_hooks_updated.png">

<!--
*Loïc*

data_table component replaces the default component
-->

---
layout: two-cols
---

Overview of the new DataTableComponent

```php {all|5,8,11,14,17}
#[AsLiveComponent(name: 'sylius_grid_data_table')]
final class DataTableComponent
{
    #[LiveProp(writable: true)]
    public string|null $grid = null;

    #[LiveProp(writable: true)] 
    public int $page = 1;
    
    #[LiveProp(writable: true)]
    public array|null $criteria = null;

    #[LiveProp(writable: true)]
    public array|null $sorting = null;

    #[LiveProp(writable: true)]
    public int|null $limit = null;
}
```

::right::

Transform your grid into a Live Component

```yaml {none|all|7-11}
sylius_twig_hooks:
    hooks:
        'sylius_admin.book.index.content.grid':
            data_table:
                component: 'sylius_grid_data_table'
                props:
                    grid: '@=_context.grid'
                    page: '@=_context.page'
                    criteria: '@=_context.criteria'
                    sorting: '@=_context.sorting'
                    limit: '@=_context.limit'                    
```

<!--
*Loïc*

another benefit : details page
just another lego piece

@= "at equals" signals that we use Expression Language syntax

_context is a Twig variable which contains all ... twig "variables"

=> TODO : find out more !!!!!!!!!!!
-->

---

<video width="700" controls autoplay loop>
  <source src="/drivers_with_component.webm">
</video>

<!--
*Loïc*

Live demo of our Live Component grid
Pagination : Change pages without refreshing the whole page / URL 
Number of items / rows to display
-->

---

## Use it in any template

Including your grid in a details page.

```twig {all|4-6}
<!-- templates/session/show/body.html.twig -->
{{ component('sylius_grid_data_table', {
    grid: 'driver',
    criteria: {
        session: session.id,
    },
}) }}
```

<!--
*Loïc*

Another benefit of using live data grid component is you can use it in any page, including details pages.

Another lego piece

Indicate the grid name + criteria for filtering

Craïtiiiiriiiiaaaaaa
-->

---

<video width="800" controls autoplay loop>
  <source src="/session_details.webm">
</video>

<!--
*Loïc*

Here is a Live details page of Session race

Drivers of the current session : we embark a prefiltered grid inside a details page with pagination and so on
-->

---
layout: center
---

Grids and Filters as Live Components

<video width="800" controls autoplay loop>
  <source src="/filters.webm">
</video>

<!--
*Loïc*

Live component filters
Each filter type needs a specific live component
- country "custom" => select

Still experimental 

TODO: check code related to live filter component !!!!
-->

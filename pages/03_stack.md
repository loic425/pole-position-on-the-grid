---
layout: image
image: /empty_dashboard.png
backgroundSize: contain
hideInToc: true
transition: fade
---

<!--
*Loïc*

After a fresh install of all the Sylius stack packages, we have this ready to use admin dashboard.
-->

---
layout: image
image: /stack_dashboard.png
backgroundSize: contain
hideInToc: true
---

<!--
*Loïc*

Here is the admin dashboard from the test application inside the mono-repository on Github.
-->

---
layout: image-right
image: /mercedes_sketch.png
backgroundSize: contain
---

# What's under the hood of the Sylius Stack?

Create a lean & mean back-office in no time with :

<v-clicks>

* <span v-mark="{ at: 5, color: 'red', type: 'circle' }">**Sylius Grid bundle**</span>
* Sylius Resource bundle
* Doctrine ORM & DBAL drivers
* Providers/processors system
* Bootstrap Admin UI
* Symfony UX, AssetMapper and more !

</v-clicks>

<!--
*Loïc*

Sylius Stack is a set of autonomous packages which, put together, let you build beautiful admin panels very quickly.

The grid bundle is a package to render list of data, it's super easy to introduce new filters, columns and customize rendering of every single part.

The resource bundle allows you to configure operations for resources, which can be, for example, a Doctrine entity.

Doctrine ORM and DBAL drivers for Rapid application development.
But now we have a Providers/Processors system which means we don't need Doctrine Entities.

On this talk, we will focus on the Grid package.
-->

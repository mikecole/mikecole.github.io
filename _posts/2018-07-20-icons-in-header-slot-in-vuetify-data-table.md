---
layout: post
title: Icons In Header Slot In Vuetify Data-Table
date: 2018-07-20
tags: vue vuetify
meta-description: When customizing the grid headers in the Vuetify `data-table`, I ran into an issue with icons too eagerly spinning. This is how I solved it.
---
<p>
We've been using <a href="https://vuetifyjs.com">Vuetify</a> for a component framework in the Vue project we've been working on the past few months. It's a great framework and
I highly recommend it, especially if you're trying to follow Material Design patterns. Today I ran into an issue customizing a grid header that was difficult to solve.
</p>

<p>
Here's the example on CodePen: <a href="https://codepen.io/mikecole/pen/zLNKbG">https://codepen.io/mikecole/pen/zLNKbG</a>
</p>

<p>
If you check the header of any column, you can see that I added a second v-icon that will eventually pop open a dropdown menu. However, when sorting the column Vuetify doesn't
distinguish that column from the arrow sorting icon, and it spins both of them.
</p>

<p>
It turns out that there doesn't seem to be any options to disable this, and that the solution seems to be fixing it with stying:
```
...
<v-icon class='no-rotate' small>filter_list</v-icon>
...
.v-datatable thead th.column.sortable.active.desc .no-rotate {
    transform: none !important;
}
```
</p>

<p>
Fixed example on CodePen: <a href="https://codepen.io/aldarund/pen/yqgVgy">https://codepen.io/aldarund/pen/yqgVgy</a>
</p>
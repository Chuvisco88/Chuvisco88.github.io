---
layout: post
title: Angular2 Container
---

##TL;DR

Use `ng-container` in combination with `ngIf` for elements which should not be displayed in the DOM.

##Long version

While doing a bit of refactoring my IDE complained about an `Unresolved variable or type tab`.

This is an obviously simple problem when looking at the code:
```angular2html
<ul>
    <li>Tab A</li>
    <li>Tab B</li>
</ul>

<div *ngIf="tabs.length > 0">
    <div class="tab-pane"
         id="{{tab.id}}"
         role="tabpanel"
         *ngFor="let tab of tabs"
         [ngClass]="{'active': tab.active}">
        <p [innerHTML]="tab.content"></p>
    </div>
</div>
```

As you might see, the variable `tab` is defined on the same element where it is used.

Therefore all that needs to be done is to move the `ngFor` one level up.

**BUT** on the level one up there is already an `ngIf` and from various sources you might now, that
it isn't possible to have multiple structural directives on the same element.
`ngIf` as well as `ngFor` are such structural directives.

One might be tempted to create something as the following:
```angular2html
[...]
<div *ngIf="tabs.length > 0">
    <div *ngFor="let tab of tabs">
        <div class="tab-pane"
             id="{{tab.id}}"
             role="tabpanel"
             [ngClass]="{'active': tab.active}">
            <p [innerHTML]="tab.content"></p>
        </div>
    </div>
</div>
```

Let me tell you: **BAD IDEA**
 
This way your DOM will be cluttered with a lot of `div`s for "nothing".

Angular2 has an interesting element called `ng-container` which itself does not get stamped to the
DOM.

We will look into that by code:
```angular2html
<ul>
    <li>Tab A</li>
    <li>Tab B</li>
</ul>

<ng-container *ngIf="tabs.length > 0">
    <ng-container *ngFor="let tab of tabs">
        <div class="tab-pane"
             id="{{tab.id}}"
             role="tabpanel"
             [ngClass]="{'active': tab.active}">
            <p [innerHTML]="tab.content"></p>
        </div>
    </div>
</ng-container>
```

This will result with a `div` per tab in your tabs property.

The result in the DOM would look something like this:
```angular2html
<ul>
    <li>Tab A</li>
    <li>Tab B</li>
</ul>

<div class="tab-pane" role="tabpanel" id="tab_a">
    <p>Tab A</p>
</div>
<div class="tab-pane" role="tabpanel" id="tab_b">
    <p>Tab B</p>
</div>
```

Whereas the code from the original problem would render to something like this:
```angular2html
<ul>
    <li>Tab A</li>
    <li>Tab B</li>
</ul>

<div>
    <div class="tab-pane" role="tabpanel" id="tab_a">
        <p>Tab A</p>
    </div>
    <div class="tab-pane" role="tabpanel" id="tab_b">
        <p>Tab B</p>
    </div>
</div>
```

Depending on your needs you might prefer the result from the original problem but then you would
still have the problem of the unresolved variable.

So to achieve the exact same result with fixing the problem would mean to
* move the `ngFor` a level up into a separate element
* make either the element with the `ngIf` or the `ngFor` to a `ng-container`

Hope this helps you :-)
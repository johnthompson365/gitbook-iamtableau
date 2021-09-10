---
description: First steps into Okta dev world
---

# Notes: Okta OAuth Integration

Starting points:

{% embed url="https://angular.io/start" %}

Angular Copied Notes:

You build Angular applications with components. Components define areas of responsibility in the UI that let you reuse sets of UI functionality.

A component consists of three things:

* **A component class** that handles data and functionality.
* **An HTML template** that determines the UI.
* **Component-specific styles** that define the look and feel.

With `*`[`ngFor`](https://angular.io/api/common/NgForOf), the `<div>` repeats for each product in the list.

\*ngFor is a Structural directives which shape or reshape the DOM's structure, by adding, removing, and manipulating elements.

`<div *ngFor="let product of products">`

Interpolation `{{ }}` lets you render the property value as text.

`{{ product.name }}`

the `<a>` element around `{{ product.name }}` ****creates a hyperlink

Property binding `[ ]` lets you use the property value in a template expression. eg. \[Title\]

`<a [title]="product.name + ' details'">`

On a `<p>` element, use an `*`[`ngIf`](https://angular.io/api/common/NgIf) directive so that Angular only creates the `<p>` element if the current product has a description.

 `<p *`[`ngIf`](https://angular.io/api/common/NgIf)`="product.description"> Description: {{ product.description }} </p>`

Bind the button's `click` event to the `share()` method in `product-list.component.ts`

 `<button (click)="share()"> Share </button>`

The share method is:

`share() { window.alert('The product has been shared!'); }`

Creating a new component generates starter files for the three parts of the component:

* `product-alerts.component.ts`
* `product-alerts.component.html`
* `product-alerts.component.css`

The `@`[`Component`](https://angular.io/api/core/Component)`()` decorator indicates that the following class is a component. `@`[`Component`](https://angular.io/api/core/Component)`()` also provides metadata about the component, including its selector, templates, and styles.

By convention, Angular component selectors begin with the prefix `app-`, followed by the component name.

The `@`[`Component`](https://angular.io/api/core/Component)`()` definition also exports the class, `ProductAlertsComponent`, which handles functionality for the component.

To make a new component available to other components in the app, in this case `ProductAlertsComponent` , add it to `AppModule`'s declarations in `app.module.ts`



{% embed url="https://developer.okta.com/docs/guides/sign-into-spa/angular/before-you-begin/" %}

git clone sample okta Angular app repo

[Installing Node.js and NPM on mac](https://treehouse.github.io/installation-guides/mac/node-mac.html) - COMPLETE




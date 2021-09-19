# Directives

22. Creating a custom attribute directive
23. @HostListener and host Object
24. Understanding @Hostbinding
25. Use the exportAs property with template refs
26. Creating a custom structural Directive

# Project `directives`

## Create new project

```bash
$ ng generate application directives -s -t --style=scss --routing=true
```
<br>

Add script to start the application

`package.json`
```json
    "start:dir": "ng serve --project directives",
```
<br>

## Initial Files

 - `app/demo/demo-credit-card.component.ts`
 - `app/demo/demo-my-for.component.ts`
 - `app/shared/credit-card.directive.ts`
 - `app/shared/tooltip.directive.ts`
 - `app/shared/my-for.directive.ts`
 - `app/shared/shared.module.ts`
 - `app/app.component.ts`
 - `app/app.module.ts`

<br>

## DemoCreditCardComponent

`app/demo/demo-credit-card.component.ts`
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'demo-credit-card',
  template: `
    <h1>Demo Credit Card Directive</h1>
    <div>
      <label>
        Credit Card Number
        <input
          name="credit-card"
          type="text"
          placeholder="Enter your 16-digit card number"
          >
      </label>
    </div>
  `
})

export class DemoCreditCardComponent implements OnInit {
  constructor() { }

  ngOnInit() { }
}

```
<br>

## DemoMyForComponent

`app/demo/demo-my-for.component.ts`
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'demo-myfor',
  template: `
    <h1>Demo Custom Structural Directive</h1>
    <div>
      <ul>
        <li *ngFor="let item of items; let i = index">
          {{i}} - {{ item.name }} - {{ item.location }}
        </li>
      </ul>
    </div>
  `
})

export class DemoMyForComponent implements OnInit {
  items = [
    { name: 'Carlos', age: 40, location: 'Zaragoza' },
    { name: 'Ana', age: 25, location: 'Madrid' },
    { name: 'Juan', age: 40, location: 'Barcelona' }
  ]

  constructor() { }

  ngOnInit() { }
}
```
<br>

## CreditCardDirective

`app/shared/credit-card.directive.ts`
```ts
import { Directive, ElementRef } from '@angular/core';

@Directive({ selector: '[credit-card]' })
export class CreditCardDirective {
  constructor(private element: ElementRef) {
    console.log(this.element);
  }
}
```
<br>

## TooltipDirective

`app/shared/tooltip.directive.ts`
```ts
import { Directive } from '@angular/core';

@Directive({ selector: '[tooltip]' })
export class TooltipDirective {
  constructor() { }
}
```
<br>

## MyForDirective

`app/shared/my-for.directive.ts`
```ts
import { Directive } from '@angular/core';

@Directive({ selector: '[myfor]' })
export class MyForDirective {
  constructor() { }
}
```
<br>

## SharedModule

`app/shared/shared.module.ts`
```ts
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';

import { CreditCardDirective } from './credit-card.directive';
import { MyForDirective } from './my-for.directive';
import { TooltipDirective } from './tooltip.directive';

@NgModule({
  declarations: [
    CreditCardDirective,
    TooltipDirective,
    MyForDirective
  ],
  imports: [
    CommonModule
  ],
  exports: [
    CreditCardDirective,
    TooltipDirective,
    MyForDirective
  ],
  providers: [],
})
export class SharedModule { }
```
<br>

## AppComponent

`app/app.component.ts`
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <demo-credit-card></demo-credit-card>
  `,
  styles: [``]
})
export class AppComponent {
  title = 'directives';
}
```
<br>


## AppModule

`app/app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { SharedModule } from './shared/shared.module';

import { DemoCreditCardComponent } from './demo/demo-credit-card.component';
import { DemoMyForComponent } from './demo/demo-my-for.component';

@NgModule({
  declarations: [
    AppComponent,
    DemoCreditCardComponent,
    DemoMyForComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    SharedModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
<br>

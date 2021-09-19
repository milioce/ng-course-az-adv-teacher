# Advanced Components

https://github.com/ultimatecourses/angular-pro-src

11. Dynamic components with ComponentFactoryResolver
12. Dynamic component @Input data
13. Dynamic component @Output subscription
14. Destroying dynamic components
15. Dynamic components reordering
16. Dynamic `<ng-template>` rendering with ViewContainerRef
17. Passing context to a dynamic `<ng-template>`
18. Dynamic `<ng-template>` rendering with `ngTemplateOutlet`
19. Using `ngTemplateOutlet` with context

=> Components 3.md

<br>

# Paso 0. Situación inicial

Tenemos una aplicacion `dynComponents` generado con Angular CLI

``` bash
$ ng generate application dynComponents --style=scss --routing=true
$ npm run start:dyncomp
```
<br>

Donde tenemos ya estos ficheros
- `app/auth-form/auth-form.component.ts`
- `app/auth-form/auth-form.interface.ts`
- `app/auth-form/auth-form.module.ts`
- `app/app.component.ts`
- `app/app.module.ts`
- `demo/demo.module.ts`
- `demo/dynamic.component.ts`
- `demo/template.component.ts`
- `demo/template-outlet.component.ts`
- `package.json`

<br>

`app/auth-form/auth-form.component.ts`
``` ts
import { Component, Output, EventEmitter } from '@angular/core';

import { User } from './auth-form.interface';

@Component({
  selector: 'auth-form',
  styles: [`
  `],
  template: `
    <div>
      <form (ngSubmit)="onSubmit(form.value)" #form="ngForm">
        <h3>{{ title }}</h3>
        <label>
          Email address
          <input type="email" name="email" ngModel #email>
        </label>
        <label>
          Password
          <input type="password" name="password" ngModel>
        </label>
        <button type="submit">
          {{ title }}
        </button>
      </form>
    </div>
  `
})
export class AuthFormComponent {
  title = 'Login';

  @Output() submitted: EventEmitter<User> = new EventEmitter<User>();

  onSubmit(value: User) {
    this.submitted.emit(value);
  }
}
```
<br>

`app/auth-form/auth-form.interface.ts`
```ts
export interface User {
  email: string,
  password: string
}
```
<br>

`app/auth-form/auth-form.module.ts`
```ts
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';

import { AuthFormComponent } from './auth-form.component';

@NgModule({
  declarations: [
    AuthFormComponent,
  ],
  imports: [
    CommonModule,
    FormsModule
  ],
  exports: [
    AuthFormComponent,
  ]
})
export class AuthFormModule {}
```
<br>

`app/app.component.ts`
``` ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <h1>{{ title }}</h1>
    <div>
      <div #entry></div>
    </div>
  `
})
export class AppComponent {
  title = 'Application: Dynamic Components';
}
```
<br>

`app/app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AuthFormModule } from 'projects/components/src/app/auth-form/auth-form.module';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    AuthFormModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
<br>

`package.json`
```json
  "scripts": {
    ...
    "start": "ng serve",
    "start:comp": "ng serve --project components",
    "start:dyncomp": "ng serve --project dynComponents",
    ...
  },
```
<br>

# Paso 11. Dynamic components with ComponentFactoryResolver

Activamos en el `app.component.ts` la demo `app-template-outlet`

`app/app.component.ts`
``` html
    <h1>{{ title }}</h1>
    <app-dynamic></app-dynamic>
```
<br>

Vamos a inyectar el `<auth-form>` de forma dinámica en el `AppCcomponent` en la referencia `#entry`


`app/demo/dynamics.component.ts`
``` html
<div>
  <div #entry></div>
</div>
```
<br>

> Vamos a usar el ViewChild para acceder al DOM del div
> - Usamos el segundo parametro {read: ViewContainerRef}

```ts
export class DynamicComponent {
  title = 'Application: Dynamic Components';

  @ViewChild('entry', {read: ViewContainerRef}) entry: ViewContainerRef;
}
```
<br>

> Inyectamos en el constructor `ComponentFactoryResolver`

`app/demo/dynamics.component.ts`
```ts
import { Component, ComponentFactoryResolver, ViewChild, ViewContainerRef } from '@angular/core';

export class DynamicComponent {
  title = 'Application: Dynamic Components';

  @ViewChild('entry', {read: ViewContainerRef}) entry: ViewContainerRef;

  constructor(private resolver: ComponentFactoryResolver) {}
}
```
<br>

> Creamos un `componentFactory` para crear instancias de nuestro componente
>  -  Tenemos que hacerlo en el `AfterViewInit`

`app/demo/dynamics.component.ts`
``` ts
export class DynamicComponent implements AfterViewInit {
  ...
  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
  }
}
```
<br>

> Tenemos que crear el componente y añadirlo el viewContainerRef

`app/demo/dynamics.component.ts`
``` ts
  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
+   const component = this.entry.createComponent(authFormFactory);
  }
```
<br>

> Tenemos que registrar el componente dinamico en el entryComponents
> del módulo `AuthFormModule`.
>  - Desde Angular 9.0 con Ivy ya no es necesario.
>  - Hacía falta cuando usamos un componente no declarado en ningún HTML

``` ts
@NgModule({
  declarations: [
    AuthFormComponent
  ],
  ...,
+ entryComponents: [
+   AuthFormComponent
+ ]
})
export class AuthFormModule {}
```
<br>
<br>

El `DynamicComponent` quedará así

`app/demo/dynamics.component.ts`
``` ts
...
export class DynamicComponent implements AfterViewInit {
  @ViewChild('entry', {read: ViewContainerRef}) entry: ViewContainerRef;

  constructor(private resolver: ComponentFactoryResolver) {
  }

  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
    const component = this.entry.createComponent(authFormFactory);
  }
}
```
<br>


# Paso 12. Dynamic component @Input data

Vamos a modificar la propiedad title del componente, no necesitamos que sea un Input.
- Usando `component.instance` podemos sobreescribir cualquier propiedad pública.

`app/demo/dynamics.component.ts`
``` ts
  ...
  constructor(
    private resolver: ComponentFactoryResolver,
    private cd: ChangeDetectorRef
  ) {
  }

  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
    const component = this.entry.createComponent(authFormFactory);
    component.instance.title = 'Create new user';
    this.cd.detectChanges();
  }

```
<br>

Al modificar la propiedad title después de inicializar la vista, nos aparece el error de *`Expression has changed after it was checked`*.

- Necesitamos inyectar el servicio `ChangeDetectorRef`
- y forzar la detección de cambios con `this.cd.detectChanges()`.

Tambien tenemos el servicio disponible en `component`
- `component.changeDetectorRef.detectChanges();`

<br>

# Paso 13. Dynamic component @Output subscription

El output es una propiedad pública del componente.
- Podemos subscribirnos a ella.

`app/demo/dynamics.component.ts`
``` ts
  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
    const component = this.entry.createComponent(authFormFactory);

    component.instance.title = 'Create new user';
    this.cd.detectChanges();

    // component.changeDetectorRef.detectChanges();

+   component.instance.submitted.subscribe(this.loginUser);
  }
```
<br>


# Paso 14. Destroying dynamic components

Añadimos un botón para destruir el componente.

`app/demo/dynamics.component.ts`
``` html
<button type="button" (click)="destroyComponent()">Destroy</button>
```
<br>

Guardamos el componente en una propiedad de la clase y accedemos a el para destruirlo.

`app/demo/dynamics.component.ts`
``` ts
import { ..., ComponentRef, ... } from '@angular/core';

  ...

  @ViewChild('entry', {read: ViewContainerRef}) entry: ViewContainerRef;
+  component: ComponentRef<AuthFormComponent>;

  ...

  ngAfterViewInit() {
    const authFormFactory = this.resolver.resolveComponentFactory(AuthFormComponent);
+   this.component = this.entry.createComponent(authFormFactory);

+   this.component.instance.title = 'Create new user';
    this.cd.detectChanges();

+   // this.component.changeDetectorRef.detectChanges();

+    this.component.instance.submitted.subscribe(this.loginUser);

+    console.log(this.component);
  }

+ destroyComponent() {
+   this.component.destroy();
+ }

```
<br>

# Paso 15. Dynamic components reordering

Al inyectar varios componentes se posicionan en orden secuencial.

`app/demo/dynamics.component.ts`
``` ts
  ngAfterViewInit() {
    ...

    // Primer componente sin modificar title
    this.entry.createComponent(authFormFactory);

    // Segundo componente
    this.component = this.entry.createComponent(authFormFactory);
    ...
  }
```
<br>

El segundo parámetro de `createComponent()` nos permite especificar el orden

`app/demo/dynamics.component.ts`
``` ts
  ngAfterViewInit() {
    ...
    // Primer componente sin modificar title
    this.entry.createComponent(authFormFactory);

    // Segundo componente, insertar en la posición 0
    this.component = this.entry.createComponent(authFormFactory, 0);
    ...
  }
```
<br>

Añadir un botón para mover el componente.

`app/demo/dynamics.component.ts`
``` html
<div>
  <button type="button" (click)="destroyComponent()">Destroy</button>
  <button type="button" (click)="moveComponent()">Move</button>
</div>
<div #entry></div>
```
<br>

Podemos mover un componente con el método `this.entry.move()`
- El primer parámetro es una referencia de la vista
- El segundo la posición

`app/demo/dynamics.component.ts`
``` ts
  moveComponent() {
    // Muevo la vista de la posición 0 (actual) a la posición 1
    this.entry.move(this.component.hostView, 1);
  }
```
<br>

Los métodos de `ViewContainerRef` nos permitirían también mover un componente hijo a otro padre
- Sacamos el componente hijo con `entry.detach()`
- y lo insertamos en el nuevo padre con `entry.insert()`

<br>

# Paso 16. Dynamic `<ng-template>` rendering with ViewContainerRef

> Activamos esta demo `app-template` en el `app.component.ts`

`app/app.component.ts`
``` ts
  template: `
    <h1>{{ title }}</h1>
    <app-template></app-template>
  `
```
<br>

Ponemos un `ng-template` en la plantilla.

`app/demo/template.component.ts`
``` html
<div #entry></div>
<ng-template #tmpl>Emilio</ng-template>
```
<br>

> Vamos a inyectar el `ng-template` dentro del `div #entry`.
> - En lugar de usar el `createComponent` usamos el `createEmbeddedView`.

`app/demo/template.component.ts`
``` ts
export class TemplateComponent implements AfterViewInit {
  @ViewChild('entry', {read: ViewContainerRef}) entry: ViewContainerRef;
  @ViewChild('tmpl') tmpl: TemplateRef<any>;

  ngAfterViewInit() {
    this.entry.createEmbeddedView(this.tmpl);
  }
}
```
<br>

Este es el método que usan las directivas estructurales como `*ngIf` y `*ngFor`.
Todas ellas usan en realidad un `ng-template`.

``` html
<div *ngIf="data">{{name}}</div>

<ng-template [ngIf]="data">
  <div>{{name}}</div>
</ng-template>
```

``` html
<div *ngFor="let item of items">{{item.name}}</div>

<ng-template ngFor let-item [ngForOf]="items">
  <div>{{item.name}}</div>
</ng-template>
```
<br>


# Paso 17. Passing context to a dynamic `<ng-template>`

Podemos pasarle un contexto al template.

`app/demo/template.component.ts`
``` html
<ng-template #tmpl let-name let-location="location">
  {{name}}, {{location}}
</ng-template>
```
- `let-location` define una variable `location` dentro de la plantilla y `="location"` define como se llama en el contexto.
- `let-name` no define un nombre especifico para el contexto, se utilizará la palabra reservada `$implicit`

<br>

Al invocar el método `createEmbeddedView()`, en el segundo parámetro le pasamos el contexto.

`app/demo/template.component.ts`
``` ts
  ngAfterViewInit() {
    this.entry.createEmbeddedView(this.tmpl, {
      $implicit: 'Emilio',
      location: 'Zaragoza'
    });
  }
```
<br>

Como el primer parámetro es `$implicit` en la plantilla podemos poner cualquier cosa:

``` html
<ng-template #tmpl let-foo let-location="location">
  {{foo}}, {{location}}
</ng-template>

```
<br>

# Paso 18. Dynamic `<ng-template>` rendering with `ngTemplateOutlet`

<br>

Activamos en el `app.component.ts` la demo `app-template-outlet`

`app/demo/template.component.ts`
``` html
    <h1>{{ title }}</h1>
    <app-template-outlet></app-template-outlet>
```
<br>

En la plantilla usamos un `ng-container` y con la directiva `ngTemplateOutlet` le indicamos que plantilla queremos inyectar en ese contenedor.

`app/demo/template.component.ts`
``` html
<ng-container [ngTemplateOutlet]="tmpl"></ng-container>

<ng-template #tmpl>
  Emiio, Zaragoza
</ng-template>
```
<br>

# Paso 19 Using `ngTemplateOutlet` with context

> Definimos el `context` para el `ng-template`

`app/demo/template.component.ts`
``` html
<ng-template #tmpl let-name let-location="location">
  {{name}}, {{location}}
</ng-template>
```
<br>

> Y definimos una propiedad `ctx` en el componente.

`app/demo/template.component.ts`
``` ts
...
export class TemplateOutletComponent  {
  ctx = {
    $implicit: 'Emilio',
    location: 'Zaragoza - Spain'
  }
}
```
<br>

> Ahora ya podemos usar la plantilla con un contexto.

`app/demo/template.component.ts`
``` html
<ng-container
  [ngTemplateOutlet]="tmpl"
  [ngTemplateOutletContext]="ctx">
</ng-container>

<ng-template #tmpl let-name let-location="location">
  {{name}}, {{location}}
</ng-template>
```
<br>

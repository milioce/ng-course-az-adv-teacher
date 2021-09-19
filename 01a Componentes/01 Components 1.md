# Advanced Components

https://github.com/ultimatecourses/angular-pro-src

1. Content projection with ng-content
2. Using ng-content with projection slots
3. Projecting and binding to components
4. @ContentChild and ngAfterContentInit
5. @ContentChildren and QueryList
6. @ViewChild and ngAfterViewInit
7. @ViewChildren and QueryList
8. @ViewChild and template #refs
9. Using ElementRef and nativeElement
10. Using platform agnostic Renderer
11. Dynamic components with ComponentFactoryResolver

<br>

# 0. Situación inicial.

<br>

`app/auth-form/auth-form.component.ts`
``` ts
import { Component, Output, EventEmitter } from '@angular/core';

import { User } from './auth-form.interface';

@Component({
  selector: 'auth-form',
  template: `
    <div>
      <form (ngSubmit)="onSubmit(form.value)" #form="ngForm">
        <h3>My Form</h3>
        <label>
          Email address
          <input type="email" name="email" ngModel>
        </label>
        <label>
          Password
          <input type="password" name="password" ngModel>
        </label>
        <button type="submit">
          Submit
        </button>
      </form>
    </div>
  `
})
export class AuthFormComponent {

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

`app/app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { AuthFormModule } from './auth-form/auth-form.module';

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

`app/app.component.ts`
```ts
import { Component } from '@angular/core';
import { User } from './auth-form/auth-form.interface';

@Component({
  selector: 'app-root',
  template: `
    <div>
      <auth-form
        (submitted)="createUser($event)">
        <h3>Create account</h3>
      </auth-form>
      <auth-form
        (submitted)="loginUser($event)">
        <h3>Login</h3>
      </auth-form>
    </div>
  `
})
export class AppComponent {

  createUser(user: User) {
    console.log('Create account', user);
  }

  loginUser(user: User) {
    console.log('Login', user);
  }

}
```
<br>


# Paso 1. Content projection with ng-content

Vamos a proyectar el contenido al formulario

- En el primer `auth-form` se proyectará `<h3>Registro</h3>`
- En el segundo `auth-form` se proyectará `<h3>Login</h3>`

`app/app.component.ts`
```html
<div>
  <auth-form (submitted)="createUser($event)">
     <h3>Register</h3>
  </auth-form>
  <auth-form (submitted)="loginUser($event)">
     <h3>Login</h3>
  </auth-form>
</div>
```
<br>

Sustituimos el h3 por <ng-content> para que se proyecte

`app/auth-form/auth-form.component.ts`
```ts
  template: `
    <div>
      <form (ngSubmit)="onSubmit(form.value)" #form="ngForm">
-       <h3>My Form</h3>
+       <ng-content></ng-content>
        ...
    </div>
  `
```
<br>

# Paso 2. Using ng-content with projection slots

Vamos a poner diferente bottón en el form

- Añadimos un `button` debajo del `<h3>`

`app/app.component.ts`
```html
  <auth-form (submitted)="createUser($event)">
     <h3>Register</h3>
     <button type="submit">Join us</button>
  </auth-form>
  <auth-form (submitted)="loginUser($event)">
     <h3>Login</h3>
     <button type="submit">Login</button>
  </auth-form>
```

- Ahora se proyecta el `h3` y el `button` en el mismo sitio porque solo tenenemos un `ng-content`
- Necesitamos decirle donde queremos que se proyecte cada cosa, utilizando un selector

`app/auth-form/auth-form.component.ts`
```html
      <form (ngSubmit)="onSubmit(form.value)" #form="ngForm">
        <ng-content select="h3"></ng-content>
        <label>
          Email address
          <input type="email" name="email" ngModel>
        </label>
        <label>
          Password
          <input type="password" name="password" ngModel>
        </label>
        <ng-content select="button"></ng-content>
      </form>

```
<br>

- En este caso se proyectara el h3 por un lado y el button por otro

- podemos usar como selectores: select="h3", select=".myClass", select="#1"

<br>

# Paso 3. Projecting components

Podemos proyectar otros componentes.

> Ya tenemos el componente que queremos proyectar `AuthRememberComponent`

`app/auth-form/auth-remember.component.ts`
```ts
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'auth-remember',
  template: `
    <label>
      <input type="checkbox" (change)="onChecked($event.target.checked)">
      Keep me logged in
    </label>
  `
})
export class AuthRememberComponent {

  @Output() checked: EventEmitter<boolean> = new EventEmitter<boolean>();

  onChecked(value: boolean) {
    this.checked.emit(value);
  }

}
```
<br>

Y registrado en `AuthFormModule`

`app/auth-form/auth-form.module.ts`
``` diff
@NgModule({
  declarations: [
    AuthFormComponent,
+   AuthRememberComponent
  ],
  imports: [
    ...
  ],
  exports: [
    AuthFormComponent,
+   AuthRememberComponent
  ]
})
export class AuthFormModule {}
```
<br>

Añadimos el componente que queremos proyectar

`app/app.component.html`
```html
<div>
  <auth-form (submitted)="createUser($event)">
     <h3>Register</h3>
     <button type="submit">Join us</button>
  </auth-form>
  <auth-form (submitted)="loginUser($event)">
     <h3>Login</h3>
     <auth-remember (checked)="rememberUser($event)">
      </auth-remember>
     <button type="submit">Login</button>
  </auth-form>
</div>
```
<br>

Y nos guardamos el valor emitido en el evento `checked`

`app/app.component.ts`
```ts
export class AppComponent {
  rememberMe = false;

  ...

  rememberUser(remember: boolean) {
    this.rememberMe = remember;
  }

}

```
<br>

> No se proyecta en ningún selector.

- Tengo que añadir <ng-content> donde proyectará todo lo que no ha encontrado un selector
- Vamos a proyectarlo con un slot

`app/auth-form/auth-form.component.ts`
```ts
  template: `
        <label>
          Password
          <input type="password" name="password" ngModel>
        </label>
+       <ng-content select="auth-remember"></ng-content>
        <ng-content select="button"></ng-content>
  `
```
<br>

# Paso 4. @ContentChild and ngAfterContentInit

- Queremos mostrar un mensaje cuando el usuario haya seleccionado el checkbox
- Necesito acceder al contenido proyectado

<br>

`app/auth-form/auth-form.component.ts`
```ts
@Component({
  selector: 'auth-form',
  template: `
   ...
        <ng-content select="auth-remember"></ng-content>
        <div *ngIf="showMessage">
          You will be logged in for 30 days
        </div>
        <ng-content select="button"></ng-content>
   ...
  `
})
export class AuthFormComponent implements AfterContentInit {
  showMessage = false;

  @ContentChild(AuthRememberComponent) remember: AuthRememberComponent;

  @Output() submitted: EventEmitter<User> = new EventEmitter<User>();

  ngAfterContentInit() {
    console.log(this.remember); // Ver propiedad publica checked

    // En la izda no proyectamos el componente y es undefined.
    if (this.remember) {

      // Necesito acceder al evento checked y cambiar el valor de showMessage
      this.remember.checked.subscribe(checked => this.showMessage = checked)
    }
  }

  ...

}
```

- Usamos `@ContentChild` para acceder al componente proyectado
- El contentido está disponible en el `ngAfterContentInit()`
- Acceso a las propiedades del componente y me susbcribo al output

<br>

# Paso 5. @ContentChildren and QueryList

- ContentChild => query para un elemento
- ContentChildren => query para varios documentos

<br>

> En el `app.component.html` repetimos el `auth-remember` varias veces

`app/app.component.html`
``` html
     ..
     <auth-remember (checked)="rememberUser($event)"></auth-remember>
     <auth-remember (checked)="rememberUser($event)"></auth-remember>
     <auth-remember (checked)="rememberUser($event)"></auth-remember>
     ..
```
- Se proyectan todos con el mismo `select`

<br>

> En el `auth-form.component.ts` accedemos a la lista de componentes

```ts
export class AuthFormComponent implements AfterContentInit {
  showMessage = false;

  @ContentChildren(AuthRememberComponent) remember: QueryList<AuthRememberComponent>;

  @Output() submitted: EventEmitter<User> = new EventEmitter<User>();

  ngAfterContentInit() {
    if (this.remember) {
      this.remember.forEach(item => {
        item.checked.subscribe(checked => this.showMessage = checked)
      });
      // this.remember.checked.subscribe(checked => this.showMessage = checked)
    }
  }

  onSubmit(value: User) {
    this.submitted.emit(value);
  }

}
```
<br>

- `QueryList` es un objeto de angular para trabajar con colecciones. Podemos ver los métodos que nos ofrece

<br>

# Paso 6. @ViewChild and ngAfterViewInit

<br>

> Quitamos los `auth-remember` repetidos y dejamos el `@ContendChild()`

<br>

> Vamos a usar el componente `auth-message`, cambiamos el mensaje por el nuevo componente.

`app/auth-form/auth-form.component.ts`
``` ts
@Component({
  selector: 'auth-form',
  template: `
        ...
        <ng-content select="auth-remember"></ng-content>
+       <auth-message
+         [style.display]="showMessage ? 'inherit' : 'none'">
+       </auth-message>
        <ng-content select="button"></ng-content>
        ...
  `
})
```
<br>

> Accedemos al componente `AuthMessageComponent` con la directiva `@ViewChild()` para modificar la propiedad `days`.
- El `ViewChild` estará disponible en el iclo `AfterViewInit`

<br>

```ts
export class AuthFormComponent implements AfterContentInit, AfterViewInit {
  showMessage = false;

  @ViewChild(AuthMessageComponent) message: AuthMessageComponent;

  ...

  ngAfterContentInit() {
    if (this.remember) {
      ...
    }
  }

  ngAfterViewInit() {
    console.log('ngAfterViewInit');
    if (this.message) {
      this.message.days = 30;
    }
  }

  ...

}
```

Si modificamos `this.message.days` dentro de `ngAfterViewInit` tenemos un error de *"Expression has changed after it was checked"*, porque estamos cambiando el valor después de que la vista haya sido inicializada.

Tenemos que definir `@ViewChild(AuthMessageComponent, {static: true})` para que este disponible antes de que angular ejecute la detección de cambios.

Y ahora ya podemos cambiar el valor dentro del `ngAfterContentInit()` y ese error desaparece.

```ts
export class AuthFormComponent implements AfterContentInit, AfterViewInit {
  ...

  @ViewChild(AuthMessageComponent, {static: true}) message: AuthMessageComponent;

  ...

  ngAfterContentInit() {
    if (this.message) {
      this.message.days = 30;
    }
    if (this.remember) {
      ...
    }
  }

  ngAfterViewInit() {
  }

  ...

}
```
<br>

# Paso 7. @ViewChildren and QueryList

Repetimos el componente `<auth-message>` varias veces y accedemos con ` @ViewChildren`.

`app/auth-form/auth-form.component.ts`
``` ts
@Component({
  selector: 'auth-form',
  template: `
        <auth-message
          [style.display]="showMessage ? 'inherit' : 'none'">
        </auth-message>
        <auth-message
          [style.display]="showMessage ? 'inherit' : 'none'">
        </auth-message>
  `
})
export class AuthFormComponent implements AfterContentInit, AfterViewInit {
  ...
  @ViewChildren(AuthMessageComponent) message: QueryList<AuthMessageComponent>;
  ...
```
<br>

Accedemos a la `queryList` dentro del `ngAfterViewInit`, pero nos vuelve a dar el error por que estamos cambiando el valor cuando la vista ha sido inicializada

`app/auth-form/auth-form.component.ts`
``` ts
  ngAfterViewInit() {
      if (this.message) {
        this.message.forEach(message => {
          message.days = 30;
        });
      }
  }
```
<br>

Una solución sería incluir el código dentro de un `setTimeout()`, pero no es la solución ideal. Funciona porque al usar `setTimeout` se difiere la ejecución a otro turno de la VM de Javascript.

Se ejecuta el código dentro del ngAfterViewInit de forma sincrona, Angular finaliza el renderizado de la vista, refleja los ultimos cambios en la plantilla y completa el turno de Javasript VM.

`app/auth-form/auth-form.component.ts`
``` ts
  ngAfterViewInit() {
    setTimeout(() => {
      if (this.message) {
        this.message.forEach(message => {
          message.days = 30;
        });
      }
    });
  }

```
<br>

Si necesitamos cambiar el estado despues de que la vista ha sido inicializada, debemos usar el servicio `changeDetector`, para que Angular vuelva a detectar cambios en modo desarrollo.


`app/auth-form/auth-form.component.ts`
``` ts
  constructor(private cd: ChangeDetectorRef) {}

  ...

  ngAfterViewInit() {
    if (this.message) {
      this.message.forEach(message => {
        message.days = 30;
      });
      this.cd.detectChanges();
    }
  }

```
<br>

# Paso 8. @ViewChild and template #refs

Añadimos una referencia en el input `#email` y accedemos a ella con un `@ViewChild`.
- `this.email` es un objeto de tipo `ElementRef`
- `this.email.nativeElement` no permite acceder al DOM (DomElement)


`app/auth-form/auth-form.component.ts`
``` ts
@Component({
  selector: 'auth-form',
  template: `
     ...
          <input type="email" name="email" ngModel #email>
     ...
  `
})
export class AuthFormComponent implements AfterContentInit, AfterViewInit {
  ...
  @ViewChild('email') email: ElementRef;
  ...
  ngAfterViewInit() {
    console.log(this.email);
    ...
  }
}
```
<br>

## 9. Using ElementRef and nativeElement

Quitamos el primer `form` del `app.component.html` y nos centramos en el Login.

`app/auth-form/auth-form.component.ts`
``` ts
  ngAfterViewInit() {
    this.email.nativeElement.setAttribute('placeholder', 'Enter your email adress');
    this.email.nativeElement.classList.add('email');
    this.email.nativeElement.focus();
    ...
  }
```
<br>

Usando la propiedad `nativeElement` podemos interactuar con el nodo específico del DOM, util si estamos programando específicamente para la web.

Angular dispone de un servicio Renderer.


# Paso 10. Using platform agnostic Renderer

Angular nos ofrece un servicio Renderer que sirve para distintos entornos y plataformas.

Para usarlo inyectamos el servicio `Renderer2` en el constructor

`app/auth-form/auth-form.component.ts`
``` ts
  ngAfterViewInit() {
    this.renderer.setAttribute(this.email.nativeElement, 'placeholder', 'Enter your email adress');
    this.renderer.addClass(this.email.nativeElement, 'email');
    this.renderer.selectRootElement(this.email.nativeElement).focus();

    // this.email.nativeElement.setAttribute('placeholder', 'Enter your email adress');
    // this.email.nativeElement.classList.add('email');
    // this.email.nativeElement.focus();
    ...
  }

```
<br>

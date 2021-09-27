# 5. Lazy loading modules

Vamos a cargar los modulos `mail` y `dashboard` mediante lazy-loading.

<br>

## 5.1 Crear módulo dashboard
---

Tenemos el módulo creado en la carpeta `app/dashboard` con dos ficheros
- `dashboard.component.ts`
- `dashboard.module.ts`

<br>

## 5.2 Creamos la ruta al módulo `dashboard`
---

En el `AppRoutingModule` añadimos una ruta con el path `dashboard`, que cargará el módulo mediante `lazy-loading`

`app/app-routing.module.ts`
```ts
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(mod => mod.DashboardModule)
  },
  {
    path: '**', redirectTo: 'folder/inbox'
  }
];

```
- Hay que definir un callback haciendo uso de `import`
- Antes se usaba un string que ahora está obsoleto
  <br> `loadChildren: './dashboard/dashboard.module#DashboardModule'`
<br>

Ahora tenemos que crear las rutas internas del módulo.

<br>

## 5.3 Crear las rutas del módulo `Dashboard`
---

Tenemos una ruta creada en el módulo

`app/dashboard/dashboard.module.ts`
``` ts
export const ROUTES: Routes = [
  {
    path: '',
    component: DashboardComponent
  }
];
@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES)
  ],
  declarations: [DashboardComponent],
})
export class DashboardModule { }
```
- El path `dashboard` que habíamos definido en el módulo `AppRoutingModule` se usa como prefijo para todas las rutas definidas en el módulo `dashboard`

<br>

## 5.4 Enlace a la ruta
---

Vamos a crear los enlaces a la ruta `dashboard`

Necesitamos modificar la plantilla `app.component.html`
- Añadimos un enlace
- Quitamos el `<mail-app>` y ponemos el `router-outlet`

`app/app.component.html`
``` html
   ...
          <a
            [routerLink]="['/dashboard']"
            routerLinkActive="active">
            Dashboard
          </a>
        </nav>
        <router-outlet></router-outlet>
      </div>
    </div>
```
<br>

Al añadir el módulo `dashboard` por lazy-loading es posible que haga fala recompilar, para que nos genere el nuevo fichero js.

Comprobar que la home no carga el módulo y al entrar a la ruta se carga el JS correspondiente.

<br>

## 5.5 Preloading lazy-loades modules
---

Por defecto cuando inicializamos el Router con `RouterModule.forRoot(routes)` no se carga ningún módulo definido como lazy-loading.

Podemos definir la estrategia que queremos:
- `NoPreloading`: Comportamiento por defecto
- `PreloadAllModules`: Precarga todos los módulos tan rápido como sea posible. Puede ser util si nuestra aplicaión tiene pocos módulos.
- `CustomPreloading`: Podemos implementar nuestra propia estrategia

<br>

`app/app-routing.module.ts`
import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

``` ts
  imports: [
    RouterModule.forRoot(routes, {preloadingStrategy: PreloadAllModules})
  ],
```
<br>

Ahora vemos que en la Home se cargan todos los módulos.

<br>

## 5.6 Custom Preloading Strategies
---

Podemos implementar una estrategia personalizada.

<br>

> Añadimos a la ruta un `data: { preload: false }`

`app/app-routing.module.ts`
``` ts
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(mod => mod.DashboardModule),
    data: { preload: false }
  },
  ...
];
```
- Vamos a hacer que se cargue el modulo inicialmente si el valor de `preload` es `true`

<br>

> Definimos una clase `customPreload`

`app/app-routing.module.ts`
``` ts
import { RouterModule, Routes, PreloadingStrategy, Route } from '@angular/router';
import { EMPTY, Observable } from 'rxjs';

class CustomPreload implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data && route.data.preload ? load() : EMPTY;
  }
}
```
- Tenemos que implementar la interface `PreloadingStrategy`
- Debemos definir un método `preload(route, load)`
  - El parámetro `route` es el objeto Route
  - `load` es una function devuelve un observable que realiza la carga.

- Si queremos que haga precarga ejecutamos el `load()` y devolvemos,
  <br>Si no queremos precarga devolvemos el observable `EMPTY`

<br>

> Definimos el `preloadingStrategy`

`app/app-routing.module.ts`
``` ts
@NgModule({
  providers: [CustomPreload],
  imports: [
    RouterModule.forRoot(routes, {preloadingStrategy: CustomPreload})
  ],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
- `CustomPreload` es un provider, necesitamos registrarlo en `providers`

<br>

> Cambiamos el valor ` { preload: false }`

Vemos que ahora en la Home no se precarga el modulo `dashboard`

<br>

## 5.7 Protecting lazy-loaded modules with canLoad
---

Vamos a usar el servicio `app/auth/auth.service` para simular autenticación

<br>

> Implementamos un servicio `AuthGuard` con un `CanLoad`

Creamos un fichero `auth/auth.guard.ts`

`app/auth/auth.guard.ts`
``` ts
import { Injectable } from '@angular/core';
import { CanLoad, Route } from '@angular/router';

@Injectable({providedIn: 'root'})
export class AuthGuard implements CanLoad {
  constructor() { }

  canLoad(route: Route) {
    return true;
  }
}
```
- Debemos implementar el interface `CanLoad`
- Debe tener un método `canLoad(route: Route)`
  - Si devuelve `true`, deja seguir la navegación
  - Si devuelve `false` la navegación se cancela

<br>

> Controlamos si tiene permisos

`app/auth/auth.guard.ts`
``` ts
@Injectable({providedIn: 'root'})
export class AuthGuard implements CanLoad {
  constructor(private authService: AuthService) { }

  canLoad(route: Route) {
    return this.authService.checkPermissions();
  }
}
```
<br>

> Redirigir navegación

Si devolvemos un objeto `urlTree`: entonces la navegación se cancela y se redirige al usuario a la nueva URL.

`app/auth/auth.guard.ts`
``` ts
import { Injectable } from '@angular/core';
import { CanLoad, Route, Router } from '@angular/router';
import { map } from 'rxjs/operators';
import { AuthService } from './auth.service';

@Injectable({providedIn: 'root'})
export class AuthGuard implements CanLoad {
  constructor(private authService: AuthService, private router: Router) { }

  canLoad(route: Route) {
    return this.authService.checkPermissions().pipe(
      map(check => check || this.redirect())
    );
  }

  private redirect() {
    return this.router.createUrlTree(['/']);
  }
}
```
- Inyectamos el servicio `Router`
- Creamos el método `redirect()`
- usamos un `pipe(map())` para devolver `true` o `redirect`

<br>

> Definimos el `canLoad` en la ruta

`app/app-routing.module.ts`
``` ts
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(mod => mod.DashboardModule),
    data: { preload: false },
    canLoad: [ AuthGuard ]
  },
  ...
];
```
<br>

Si accedemos a `/folder/trash` y hacemos click en el enlace `Dashboard`, vemos como se cancela la navegación y nos redirige al inicio.

<br>

## 5.8 Guards with canActivate
---

Primero vamos a refactorizar las rutas del módulo `MailModule`

<br>

### 5.8.1 Refactorizar rutas  `MailModule`
---

Cambiamos las rutas para definir una ruta `mail` con dos rutas hijas.

`app/mail/mail.module.ts`
``` ts
export const ROUTES: Routes = [
  {
    path: 'mail',
    component: MailAppComponent,
    children: [
      {
        path: 'folder/:name',
        component: MailFolderComponent,
        resolve: {
          messages: MailFolderResolve
        }
      },
      {
        path: 'message/:id',
        component: MailViewComponent,
        outlet: 'pane',
        resolve: {
          message: MailViewResolve
        }
      }
    ]
  }
];
```
- Ahora las rutas quedan como `"mail/folder/:name"`

<br>

Y añadimos el `"mail/"` también a la ruta del error 404

`app/app-routing.module.ts`
``` ts
  { path: '**', redirectTo: 'mail/folder/inbox' }
```
<br>

Modificamos los enlaces en la plantilla del componente `app.component.ts`

`app/app.component.ts`
```html
  <nav>
    <a
      [routerLink]="[{outlets: {primary: 'mail/folder/inbox', pane: null }}]"
      routerLinkActive="active">
      Inbox
    </a>
    <a
      [routerLink]="[{outlets: {primary: 'mail/folder/trash', pane: null }}]"
      routerLinkActive="active">
      Trash
    </a>
    ...
  </nav>

```
<br>

Modificamos el enlace a `message` en `MailItemComponent`

`app/mail/components/item/mail-item.component.ts`
```ts
  navigateToMessage() {
    this.router.navigate(
      ['/mail', { outlets: { pane: ['message', this.message.id] } }]
    );
  }
```
<br>

### 5.8.2 Añadir el guard a la ruta `"mail"`

Con `canActivate` podemos proteger una ruta y todas sus hijas.

<br>

`app/mail/mail.module.ts`
``` ts
export const ROUTES: Routes = [
  {
    path: 'mail',
    component: MailAppComponent,
    canActivate: [ AuthGuard ],
    children: [
      ...
    ]
  }
];
```
<br>

Y definimos el comportamiento de canActivate en el Guard

`app/auth/auth.guard.ts`
``` ts
export class AuthGuard implements CanLoad, CanActivate {
  ...

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return this.authService.isLoggedIn();
  }

  ...
}
```
<br>

Si modificamos `authService.isLoggedIn()` vamos que no se cargan las rutas 'mail/*'

<br>

## 5.9 Guards with canActivateChild
---

Si definimos `canActivateChild` en la ruta padre
- Tenemos acceso a la ruta padre
- Protegemos sólo las rutas hijas.


`app/mail/mail.module.ts`
``` ts
export const ROUTES: Routes = [
  {
    path: 'mail',
    component: MailAppComponent,
-   canActivate: [ AuthGuard ],
+   canActivateChild: [ AuthGuard ],
    children: [
      ...
    ]
  }
];
```
<br>

Y tenemos que implementar el interface `CanActivateChild` en el `AuthGuard`

`app/auth/auth.guard.ts`
``` ts
export class AuthGuard implements CanLoad, CanActivate, CanActivateChild {
  ...

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return this.authService.isLoggedIn();
  }

  canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return this.authService.isLoggedIn();
  }

  ...
}
```
- Ahora podemos acceder a `"/mail"`, pero no a `"/mail/*"`

<br>


## 5.10 Guards with canDeactivate

### 5.10.1  Añadir formulario enviar mensaje

Vamos a añadir un formulario de enviar mensaje en el componente `MailViewComponent`.

`app/mail/components/mail-view/mail-view.component.ts`
``` html
    <div class="mail-view" *ngIf="(message$ | async) as message">
      <h2>{{ message.from }}</h2>
      <p>{{ message.full }}</p>
    </div>
    <div class="mail-reply">
      <textarea
        (change)="updateReply($event.target.value)"
        placeholder="Type your reply..."
        [value]="reply">
      </textarea>
      <button type="button" (click)="sendReply()">
        Send
      </button>
    </div>
```
<br>

Modificamos el TS

`app/mail/components/mail-view/mail-view.component.ts`
``` ts
  reply = '';
  hasUnsavedChanges = false;

= constructor(private route: ActivatedRoute) { }
= ngOnInit() { }

  updateReply(value: string) {
    this.reply = value;
    this.hasUnsavedChanges = true;
  }

  sendReply() {
    console.log('Sent message!', this.reply);
    this.hasUnsavedChanges = false;
    this.reply = '';
  }
```
<br>

- Cuando cambio de mensaje se mantiene escrito el texto del textarea.
- Lo inicializamos cuando cambien los parametrs de la ruta.

`app/mail/components/mail-view/mail-view.component.ts`
``` ts
  ngOnInit() {
    this.route.params.subscribe(() => {
      this.reply = '';
      this.hasUnsavedChanges = false;
    });
  }
```
<br>

### 5.10.2 Definimos un servicio `MailViewGuard`

<br>

> Crear nuevo fichero `app/mail/components/view/mail-view.guard.ts`

`app/mail/components/mail-view/mail-view.guard.ts`
``` ts
import { Injectable } from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { MailViewComponent } from './mail-view.component';

@Injectable({providedIn: 'root'})
export class MailViewGuard implements CanDeactivate<MailViewComponent> {
  constructor() { }

  canDeactivate(component: MailViewComponent) {
    return true;
  }
}
```
- Implementa el interface `CanDeactivate<T>`
  <br>donde `T` es el `routed component` asociado al canDeactivate
- el método `canActivate(component: T)` recibe como parametro el componente

<br>

> Implementamos el `canDeactivate`

`app/mail/components/mail-view/mail-view.guard.ts`
``` ts
  // Si hay cambios pendientes pedimos confirmación:
  //   - devuelve True si podemos cambiar de ruta,
  //   - Devuelve False si cancelamos la navegación

  canDeactivate(component: MailViewComponent) {
    if (component.hasUnsavedChanges) {
      return window.confirm('Are you sure you want to leave?');
    }

    return true;
  }
```
<br>

Los parametros que recibe `canDeactivate` son

```ts
  canDeactivate(
    component: T,
    currentRoute: ActivatedRouteSnapshot,
    currentState: RouterStateSnapshot,
    nextState?: RouterStateSnapshot
  )
```
- `RouterStateShaphot`, representa el estado del router en un instante de tiempo.
- Es un árbol de `activated routes snapshots`. Cada nodo contiene información sobre los segmentos de URL, params y resolved data.
- `ActivatedRouteShapshot` es la información de la ruta asociada a un componente cargado en un oulet en un instante de tiempo.

<br>

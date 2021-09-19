# Indice Routing

# 1. Files
 - `mock/db.json`: new resource `messages`
 - `app/mail/pages`
 - `app/mail/components`
 - `app/mail/models`

# 5.1 Introducción

El código inicial del proyecto está en el carpeta `projects/routing`

```bash
$ npm run start:routing
```
<br>

En el `AppRoutingModule` tenemos definida una ruta.
- Cualquier patrón redirecciona a `folder/inbox`

`app/app-routing.module.ts`
```ts
const routes: Routes = [
  { path: '', redirectTo: 'folder/inbox', pathMatch: 'full' }
];
```
<br>

# 5.2 Enabling route tracing

Al importar el módulo con las rutas podemos activar el debug.
- Muestra los eventos de navegación en la consola

`app/app-routing.module.ts`
```ts
  RouterModule.forRoot(routes, { enableTracing: true })
```
<br>

Quitamos el debug.

<br>

# 5.3 Subscribing to router events

Intectamos el `Router` en el constructor y
nos suscribimos a los eventos del router

`app/app.component.ts`
``` ts
import { Router } from '@angular/router';

export class AppComponent implements OnInit {

  constructor(private router: Router) {}

  ngOnInit() {
    this.router.events.subscribe(console.log);
  }
}
```
<br>

> Ahora vamos a filtrar solo el evento `NavigationEnd`

`app/app.component.ts`
``` ts
  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationEnd) {
        console.log(event);
      }
    });
  }
```
<br>

> También podemos suscribirno solo a eventos de tipo NavigationEnd

`app/app.component.ts`
``` ts
import { filter } from 'rxjs/operators';
...

  ngOnInit() {
    this.router.events.pipe(
      filter(event => event instanceof NavigationEnd)
    )
    .subscribe(event => {
      console.log(event);
    });
  }
```
<br>


# 5.4 Router outlet events

Vamos a ver los eventos del `router-outlet`.

`app/app.component.ts`
``` ts
  template: `
    <div class="mail">
      <router-outlet
        (activate)="onActivate($event)"
        (deactivate)="onDeactivate($event)">
      </router-outlet>
    </div>
  `
})

  onActivate(event) {
    console.log('Activate:', event);
  }

  onDeactivate(event) {
    console.log('Deactivate:', event);
  }
```

- Vemos en la consola que se ha activado el componente `MailFolderComponent`
 <br> y podemos ver sus propiedades.

- El `$event` es una instancia del componente.
  <br> Podemos acceder dinámicamente a sus propiedades o suscribirnos a un ouput.

<br>

# 5.5 Dynamic route resolves with snapshots

`Resolve` precarga los datos antes de navegar a una ruta de componente.

Vamos a configurar un `resolve` para nuestro `MailFolderComponent`.

<br>

> Paso1. Tenemos el servicio `MailService` para obtener los datos

`app/mail/services/mail.service.ts`
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

import { Mail } from '../models/mail.model';

@Injectable({providedIn: 'root'})
export class MailService {
  constructor(private http: HttpClient) { }

  getFolder(folder: string): Observable<Mail[]> {
    return this.http
      .get<Mail[]>(`/api/messages?folder=${folder}`);
  }

  getMessage(id: string): Observable<Mail> {
    return this.http
      .get<Mail>(`/api/messages/${id}`);
  }

}
```
<br>


> Paso 2. Vamos a crear el resolve en `app/mail/pages/folder/mail-folder.resolver.ts`

`app/mail/pages/folder/mail-resolve.folder.ts`
```ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';
import { Mail } from '../../models/mail.model';
import { MailService } from '../../services/mail.service';

@Injectable({ providedIn: 'root' })
export class MailFolderResolve implements Resolve<Mail[]> {
  constructor(private mailService: MailService) {}

  resolve(route: ActivatedRouteSnapshot): Observable<Mail[]> {
    // The route is folder/:name
    return this.mailService.getFolder(route.params.name);
  }
}
```
<br>

> Paso 3. Definimos el resolve en la ruta, en `MailModule`

<br>

`app/mail/mail.module.ts`
```ts
export const ROUTES: Routes = [
  {
    path: 'folder/:name',
    component: MailFolderComponent,
    resolve: {
      messages: MailFolderResolve
    }
  }
];
```
<br>

> Paso 4. En el componente `MailFolderComponent` obtenemos los mensajes del router y el titulo del folder.
> - Inyectamos el servicio `ActivatedRoute`
> - El `resolve` añade los datos en el objeto `route.data`, un observable que emite los datos.
> - Modificamos la plantilla

`app/mail/xxx`
```ts
import { Component } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { pluck } from 'rxjs/operators';

@Component({
  selector: 'mail-folder',
  styleUrls: ['mail-folder.component.scss'],
  template: `
+   <h2>{{title | async}}</h2>
    <mail-item
+     *ngFor="let message of (data | async)"
      [message]="message">
    </mail-item>
  `
})
export class MailFolderComponent {

+ data = this.route.data.pipe(
+   pluck('messages')
+ );

+ title = this.route.data.pipe(
+   pluck('name')
+ );

+ constructor(private route: ActivatedRoute) {}
}
```
<br>


# 5.6 Auxiliary named routed outlets

> Limpiamos los eventos del router-outlet en el `AppComponent`

<br>

> Paso 1. Creamos un router-outlet auxiliar en el `AppComponent`,
> al mismo nivel que el principal.

`app/app.component.ts`
```html
        <div class="mail">
          <router-outlet></router-outlet>
        </div>
        <div class="mail">
          <router-outlet name="pane"></router-outlet>
        </div>
```
<br>

> Paso 2. Vamos a crear la navegación en el nuevo outlet
> - creamos una ruta en el `MailModule`

```ts
export const ROUTES: Routes = [
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
    outlet: 'pane'
  }
];

```
- Podemos verla poniendo en el navegador eta URL: `http://localhost:4200/folder/inbox(pane:message/2)`

<br>

# 5.7 Navigating to auxiliary named outlets

<br>

> Paso 1. Vamos a defnir el routerLink en cada `mail-item` a la ruta auxiliar

- En una ruta normal sería `[routerLink]="['/message', message.id]"`

- Ahora definimos en la ruta actual [''],
  <br> la navegación auxiliar `{ outlets: { pane: ['message', message.id] } }`

<br>

`app/mail/components/item/mail-item.component.ts`
``` ts
@Component({
  ...
  template: `
    <a
+     [routerLink]="['', { outlets: { pane: ['message', message.id] } }]"
+     routerLinkActive="active"
      class="mail-item">
      <h3>
        {{ message.from }}
        <span>{{ message.timestamp | date:'shortTime' }}</span>
      </h3>
      <p>{{ message.summary }}</p>
    </a>
  `
})
```
- Ver el enlace que genera al navegar, no se ve todavía el mensaje
<br>


# 5.8 Auxiliary navigation API

Vamos a ver como navegamos por una ruta auxiliar, desde TS.

<br>

> Cambiamos el `routerLink`

`app/mail/item/mail-item.component.ts`
```ts
// [routerLink]="[ '', {outlets: { pane: ['message', message.id] }} ]"

@Component({
  selector: 'mail-item',
  styleUrls: ['mail-item.component.scss'],
  template: `
    <a
+      (click)="navigateToMessage()"
      routerLinkActive="active"
      class="mail-item">
      <h3>
        {{ message.from }}
        <span>{{ message.timestamp | date:'shortTime' }}</span>
      </h3>
      <p>{{ message.summary }}</p>
    </a>
  `
})
```
<br>

> Inyectamos el servicio `Router`

`app/mail/item/mail-item.component.ts`
```ts
export class MailItemComponent {
  @Input() message: Mail;

+ constructor(private router: Router) {}

+ navigateToMessage() {
+   this.router.navigate(
+     ['', { outlets: { pane: ['message', this.message.id] } }]
+   );
+ }

}
```
- Vemos cómo va cambiando la url con la ruta auxiliar

<br>


# 5.9 Destroying auxiliary outlets

Si ahora navegamos del `inbox` al `trash` se mantiene la navegación del `outlet` auxiliar.

Necesitamos destruirlo.

<br>

> Moodificamos los `routerLink` del `AppComponent`

Necesitamos indicar la navegación de cada outlet.

`app/app.component.html`
``` html
  <nav>
    <a
-     routerLink="folder/inbox"
+     [routerLink]="[{ outlets: { primary: 'folder/inbox', pane: null } }]"
      routerLinkActive="active">
      Inbox
    </a>
    <a
-     routerLink="folder/inbox"
+     routerLink="[{ outlets: { primary: 'folder/trash', pane: null } }]"
      routerLinkActive="active">
      Trash
    </a>
  </nav>
```
<br>


# 5.10 Resolver for auxiliary outlets

Vamos a crear un `resolve` para el detalle del mensaje.

<br>

> Paso 1. Creamos el servicio `MailViewResolve`

<br>

`app/mail/components/mail-view/mail-view.resolve.ts`
``` ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot } from '@angular/router';
import { Observable } from 'rxjs';

import { Mail } from '../../models/mail.model';
import { MailService } from '../../services/mail.service';

@Injectable({ providedIn: 'root' })
export class MailViewResolve implements Resolve<Mail> {

  constructor(private mailService: MailService) {}

  resolve(route: ActivatedRouteSnapshot): Observable<Mail> {
    return this.mailService.getMessage(route.params.id);
  }
}
```
- Creamos un nuevo fichero con el esqueleto de una clase tipo `resolver`
- Inyectamos el servicio `MailService`
- Devolvemos el mensaje

<br>

> Paso 2. Definimos el `resolve` en la ruta

`app/mail/mail.module.ts`
```ts
export const ROUTES: Routes = [
  {
    ...
  },
  {
    path: 'message/:id',
    component: MailViewComponent,
    outlet: 'pane',
    resolve: {
      message: MailViewResolve
    }
  }
];
```
<br>


> Paso 3. Obtenemos los datos del `resolve` y los mostramos.

`app/mail/components/mail-view/mail-view.component.ts`
```ts
export class MailViewComponent implements OnInit {
+ message$ = this.route.data.pipe(
+   pluck('message')
+ ) as Observable<Mail>;

+ constructor(private route: ActivatedRoute) { }

  ngOnInit() { }
}

```
- Inyectar `ActivatedRoute`
- Definimos el obserbable `message$`

<br>

Y modificamos la plantilla

`app/mail/components/mail-view/mail-view.component.ts`
```html
    <div class="mail-view" *ngIf="message$ | async as message">
      <h2>{{ message.from }}</h2>
      <p>{{ message.full }}</p>
    </div>
```
<br>


<br>
---
Estoy aquí
---
---

<br>

> xxx

`app/mail/xxx`
```ts
```
<br>

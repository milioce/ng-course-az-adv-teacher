# Indice

1 Interceptor
2 Zones and NgZone

# Analisis
- Añadir un coreModule
  - Añadir servicio auth.service.ts para simular autenticacion
  - Añadimos un interceptor para añadir un header

  - Añadir un interceptor para interceptar errores


# 6.0 Initial files

> Crear `CoreModule`

- Carpeta `app/core`
- un fichero `app/core/` **`core.module.ts`**
- uan carpeta `app/core/` **`services`**
- una carpeta `app/core/` **`interceptors`**

<br>

> Creamos el servicio `auth.service.ts`

`app/core/services/auth.service.ts`
``` ts
import { Injectable } from '@angular/core';

@Injectable({providedIn: 'root'})
export class AuthService {
  private token = 'abcd-1234-5678';

  constructor() { }

  getToken(): string {
    return this.token;
  }

}
```
<br>

# 6.1 Qué es un interceptor

Un `Interceptor` intercepta la petición HTTP. la modifica si es necesario y continua su camino, pasandola al siguiente interceptor de la cadena invocando next.handle().

Para usar la misma instancia del interceptor en toda la aplicación, hay que importar el `HttpClientModule` solo en el `AppModule` y añadir el interceptor al `root application injector`.

Si incluimos el `HttpClientModule` en otros módulos, cada import crea una nueva copia que sobreescribe los interceptors.

<br>

# 6.1 Auth Interceptor

Vamos a simular autenticación en un API y enviaremos en todas las peticiones una cabecera con el token de autorización.

<br>

> Crear `auth.interceptor.ts`

Creamos un esqueleto de un `Http Interceptor`

`app/core/interceptors/auth.interceptor.ts`
``` ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpEvent, HttpHandler, HttpRequest } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler):  Observable<HttpEvent<any>> {
    return next.handle(req);
  }
}
```
- `req` El objeto Request saliente
- `next` El siguiente interceptor de la cadena o la respuesta final

<br>

> Modificamos la request

Si hay token
- clonamos la request
- Añadimos un header a los headers que ya tiene la request
- Pasamos la request modificada al siguiente de la cadena

`app/core/interceptors/auth.interceptor.ts`
``` ts
// Constante con el key de la cabecera
const HEADER_AUTH = 'Authorization';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  // Inyectamos el servicio
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Obtener el token del usuario
    const token = this.authService.getToken();

    if (token) {
      // clonamos la request y añadimos el header a los actuales
      const clonedReq = req.clone({
        headers: req.headers.set(HEADER_AUTH, 'Bearer ' + token)
      });

      // pasamos la request modificada al siguiente
      return next.handle(clonedReq)
    }

    // Pasamos la request sin modificar
    return next.handle(req);
  }
}
```
<br>

> Registrmos el `AuthInterceptor` como provider

`app/core/core.module.ts`
```ts
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ],
```
<br>

> Importamos el módulo `CoreModule` en el `AppModule`

Ya podemos ver la cabecera en la petición que se hace a `products`.

Cambiamos en el `AppComponent` a `app-food` y vemos que aparece la cabecera en las 3 peticiones que hace.

<br>

# 6.2. Error Interceptor

## 6.2.1 Refactor
---

<br>

> Paso 1. Crear componente `ErrorComponent`

`app/core/error.component.ts`
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  template: `
    <h1>Error</h1>
    <p>Se ha producido un error</p>
  `
})

export class ErrorComponent implements OnInit {
  constructor() { }

  ngOnInit() { }
}
```
<br>

Declaramos el componente en `CoreModule`

`app/core/core.module.ts`
```ts
  declarations: [
    ErrorComponent
  ],
```
<br>

> Paso 2. Definir rutas en el `AppRoutingModule`

Definimos rutas para `food`, `store` y `error`

`app/app-routing.module.ts`
```ts
const routes: Routes = [
  { path: '', redirectTo: 'food', pathMatch: 'full' },
  { path: 'food', component: FoodComponent },
  { path: 'store', component: StoreComponent },
  { path: 'error', component: ErrorComponent },
];
```
<br>

> Poner el `router-outlet` para navegar en el `AppComponent`

Ponemos el `router-outlet` y varios botones para navegar

`app/app.component.ts`
```ts
@Component({
  selector: 'app-root',
  styles: [`
    button {
      margin: 0 10px;
    }
  `],
  template: `
    <button type="button" (click)="onTestError()">Test error</button>
    <button type="button" (click)="onNavigate('food')">Go to food</button>
    <button type="button" (click)="onNavigate('store')">Go to store</button>
    <router-outlet></router-outlet>
  `,
  styles: []
})
export class AppComponent {
  title = 'services';

  constructor(private http: HttpClient, private router: Router) {}

  onTestError() {
    this.http.get('/api/error').subscribe();
  }

  onNavigate(path) {
    this.router.navigate([path]);
  }
}

```
- Ponemos el `<router-outlet>` para navegar
- En el primer botón hacemos una peticion que va a dar un error 404

<br>


## 6.2.2 Crear ErrorInterceptor
---

Creamos el fichero `error.interceptor.ts`

`app/core/interceptors/error.interceptor.ts`
```ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpEvent, HttpHandler, HttpRequest } from '@angular/common/http';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { Router } from '@angular/router';

@Injectable()
export class ErrorInterceptor implements HttpInterceptor {

  constructor(private router: Router) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      tap(
        () => {},
        (err) => {
          if (err.status === 404) {
            this.router.navigate(['/error'])
          }
        }
      )
    );
  }
}
```
<br>


## 6.2.3 Registrar interceptor
---

Registramos el provider con el `ErrorInterceptor` en el `CoreModule`

> xxx

`app/core/core.module.ts`
```ts
@NgModule({
  imports: [],
  exports: [],
  declarations: [ ErrorComponent ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    },
    {
      provide: HTTP_INTERCEPTORS,
      useClass: ErrorInterceptor,
      multi: true
    }
  ],
})
export class CoreModule { }
```
<br>

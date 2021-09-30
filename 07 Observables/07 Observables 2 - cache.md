# 7.2 Cachear peticiones

Hay dos posibilidades:
- Cachear la respuesta
- Reutilizar el mismo observable (multicasting)

# 7.2.1 Cachear la respuesta

`app/cache/services/data.service.ts`
```ts
export class DataService {
  cache: any = null;

  getCountries() {
    return this.getCacheResponse();
  }

  getCacheResponse() {
    if (this.cache) {
      return of(this.cache);
    }

    return this.getRequest().pipe(
      tap(data => this.cache = data)
    );
  }

  ...
}
```
<br>

# 7.2.2 Cachear el observable y convertir a multicast

> Cachear el observable

`app/cache/services/data.service.ts`
```ts
  // Cachear observable => repite request porque no es multicast
  getCountries1() {
    if (!this.cache) {
      this.cache = this.getRequest();
    }

    return this.cache;
  }
```
<br>

> Convertir a multicast

`shareReplay` utiliza ìnternamente un `SubjectReplay`
- El observable interno se suscribe al origen (1 suscripcion)
- Las suscripciones se hacen al observable interno (multicas)

<br>

`app/cache/services/data.service.ts`
```ts
  // Convertimos a multicast
  getCountries2() {
    if (!this.cache) {
      this.cache = this.getRequest().pipe(
        shareReplay(1)
      );
    }

    return this.cache;
  }
```
<br>

> Ejercicio. Invalidar la cache

- Hacer que se invalide la cache automáticamente a los 10 segundos
  - Sin repetir request cada 10 seg.
  - Invalidamos y la proxima vez que alguien se suscriba volverá a lanzar la request

Partimos del código anterior

`app/cache/services/data.service.ts`
```ts
  // Convertimos a multicast
  getCountries3() {
    if (!this.cache) {
      this.cache = this.getRequest().pipe(
        shareReplay(1)
      );
    }

    return this.cache;
  }
```
<br>

Solución

`app/cache/services/data.service.ts`
```ts
  getCountries3() {
    if (!this.cache) {
      const clean$ = timer(10000).pipe(
        tap(_ => this.cache = null)
      );

      this.cache = this.getRequest().pipe(
        tap(_ => clean$.subscribe()),
        shareReplay(1),
      );
    }

    return this.cache;
  }
```
<br>

Bonus

# 7.2.3 Ejercicio Interceptor

Existen otras posibilidades para cachear

- Extender HttpClient y cachear peticiones GET (multicast o response)
- Interceptor cachear peticiones GET (response)

<br>

> Ejercicio. Interceptor

- Crear un interceptor `CacheInterceptor` que cachee todas las peticiones GET. (Cachear los datos de la respuesta)

```ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpEvent, HttpHandler, HttpRequest } from '@angular/common/http';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements HttpInterceptor {
  cache = new Map();

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    if (this.isCacheable(req)) {
      const idcache = this.getCacheId(req);
      if (this.cache.has(idcache)) {
        return of(this.cache.get(idcache));
      }

      return next.handle(req).pipe(
        tap(response => this.cache.set(idcache, response))
      );
    }

    return next.handle(req);
  }

  private isCacheable(req: HttpRequest<any>) {
    return req.method === 'GET';
  }

  private getCacheId(req: HttpRequest<any>) {
    return req.url;
  }

}
```
<br>

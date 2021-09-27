# Indice

10.1 Providers and useValue
10.2 Using inyectionToken
10.3 Providers and useClass
10.4 Providers and useFactory
10.5 Providers and useExisting
10.6 Configurable NgModules
10.7 Zones and NgZone

# 6.0 Inicio

La aplicación se ha creado con este comando:

```bash
$ ng generate application services -s -t --style=scss --routing=true
```
<br>

Vamos a usar el módulo `FoodModule`
- Tenemos 3 componentes `FoodPizzasComponent`, `FoodSidesComponent` y `FoodDrinksComponent`, que muestran datos (por ahora los mismos)
- Tenemos un servicio `FoodService` donde tenemos una variable `api`

<br>

# 6.1 Providers and useValue

Vamos a definir un `provider` en el `AppModule` con la URL del Api.
- token : "api"
- valor : "/api/pizzas"

`app/app.module.ts`
``` ts
  providers: [
    { provide: 'api', useValue: '/api/pizzas' }
  ],

```
<br>

Ahora inyectamos este token en el servicio `FoodService` para usarlo
- `@Inject(token)` especifica un `provider` personalizado de dependencia de un constructor.

<br>

`app/food/food.service.ts`
``` ts
export class FoodService {

  constructor(
    private http: HttpClient,
    @Inject('api') private api: string
  ) {}

  getFood(): Observable<any[]> {
    return this.http.get<any[]>(this.api);
  }
}
```
<br>

`constructor(private service: TestService)` es equivalente a

`constructor(@Inject(TestService) private service: TestService)`

<br>

> Pasamos el provider al módulo `FoodModule`

```ts
@NgModule({
  ...
  providers: [
    { provide: 'api', useValue: '/api/pizzas'}
  ],
})
export class FoodModule { }
```
<br>

# 6.2 Using InyectionToken

No podemos definir dos providers con el mismo string, porque el segundo sobreescribe el anterior.

``` ts
  providers: [
    { provide: 'api', useValue: '/api/pizzas' },
    { provide: 'api', useValue: '/api/fruits' }
  ],
```
<br>

## 6.2.1 Definimos un `InjectionToken`
---

Con `InjectionToken` definimos un token que podemos usar en un DI Provider

`app/food/food.token.ts`
``` ts
import { InjectionToken } from "@angular/core";

export const FOOD_API_TOKEN = new InjectionToken<string>('api');
```
<br>

Usamos ese token para definir un provider en el `AppModule`

`app/food/food.module.ts`
``` ts
import { FOOD_API_TOKEN } from './token';

  providers: [
    { provide: FOOD_API_TOKEN, useValue: '/api/pizzas' }
  ],
```
<br>

Inyectamos ese token en el servicio

`app/food.service.ts`
``` ts
import { FOOD_API_TOKEN } from './token';

  constructor(
    private http: HttpClient,
    @Inject(FOOD_API_TOKEN) private api: string
  ) {}
```
<br>

Ahora si que podríamos crear varios tokens / providers con el mismo nombre

``` ts
export const FOOD_API_TOKEN1 = new InjectionToken<string>('api');
export const FOOD_API_TOKEN2 = new InjectionToken<string>('api');

providers: [
    { provide: FOOD_API_TOKEN1, useValue: '/api/xxx' }
    { provide: FOOD_API_TOKEN2, useValue: '/api/yyy' }
  ],

```
<br>

# 6.3 Providers and useClass

### 6.3.1 Refactor
---

<br>

> Paso 1. Registramos el `FoodService` como `provider` en el módulo `FoodModule`

- Quitamos el `{providedIn: 'root'}` del fichero `food.service.ts`
- Lo añadimos en el array `providers: []` del fichero `food.nodule.ts`

`app/food/food.module.ts`
``` ts
import { FOOD_API_TOKEN } from './token';

  providers: [
    FoodService,
    { provide: FOOD_API_TOKEN, useValue: '/api/pizzas' }
  ],
```
- En este caso el módulo usa su propia instancia del servicio

<br>


> Paso 2. Regisrar el servicio como `provider` en el Componente

Quitamos el provider del módulo y lo registramos dentro de cada componente.

`app/food/components/food-pizzas.component.ts`
``` ts
@Component({
  selector: 'app-food-pizzas',
  providers: [ FoodService ],
  template: `
    ...
  `
})
```
- Hacemos lo mismo con `FoodDrinksComponent` y `FoodSidesComponent`
- En este caso cada componente usa una instancia distinta del servicio `FoodService`
  <br> Ponemos un console.log en el constructor para comprobarlo

<br>

La definición de un provider
- `providers: [ FoodService ]`

es equivalente a
- `providers: [ { provide: FoodService, useClass: FoodService } ]`

<br>

### 6.3.2 Definir un provider con una nueva clase
---

Vamos a definir un provider para `foodService ` pero que use una nueva clase, por ejemplo un mock `FoodMockService`

> Paso 1. Creamos el nuevo servicio

`app/food/food-mock.service.ts`
``` ts
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';

import { Food } from './food.service';

@Injectable()
export class FoodMockService {
  foods = [
    { name: 'Agua', price: 0.90 },
    { name: 'Cerveza', price: 1.50 },
    { name: 'Vino', price: 1 }
  ];

  constructor() {
    console.log('FoodMockService::constructor()');
  }

  getFood(): Observable<Food[]> {
    return of(this.foods)
  }

}
```
<br>

> Paso 2. Definimos el provider en el componente `FoodDrinksComponent`.

`app/food/components/food-drinks.component.ts`
``` ts
@Component({
  selector: 'app-food-drinks',
  providers: [
    // FoodService,
    { provide: FoodService, useClass: FoodMockService}
  ],
  ...
}
```
- `useClass` permite crear y devolver una nueva instancia de una clase especificada
- Se puede usar para sustituir una clase predetermina por una alternativa:
  - Implementar una estrategia diferente,
  - extender una clase,
  - emular una clase real para ejecutar pruebas
<br>

En la consola vemos que se ha creado una instancia del `FoodMockService`

<br>


# 6.4 Providers and useFactory

Podemos definir un provider con una función que se encarga de crear la instancia de la clase (factory)

`app/food/components/food-sides.component.ts`
``` ts
@Component({
  selector: 'app-food-sides',
  providers: [
    // FoodService
    {
      provide: FoodService,
      useFactory: () => new FoodMockService()
    }
  ],
  ...
}
```
- En la consola vemos como ahora el `FoodMockService` se crea dos veces

<br>

Si nuestro servicio tiene dependencias las tenemos que definir en el provider


`app/food/components/food-sides.component.ts`
``` ts
@Component({
  selector: 'app-food-sides',
  providers: [
    // FoodService
    {
      provide: FoodService,
      useFactory: (http) => new FoodService(http, '/api/sides'),
      deps: [HttpClient]
    }
  ],
  ...
})
```
- Se muestran los datos del mock `sides`

<br>

Vamos a modificar el segundo parámetro de `FoodService` y le quitamos el `@Inject`

`app/food/food.service.ts`
``` ts
export class FoodService {

  constructor(
    private http: HttpClient,
    private api: string
    // @Inject(FOOD_API_TOKEN) private api: string
  ) {
    console.log('FoodService::constructor()');
  }
  ...
}
```
<br>

Cambiamos tambien los otros dos componentes `FoodPizzasComponent` y `FoodDrinksComponent` cada uno con su api.

`app/food/components/food-pizzas.component.ts`
``` ts
@Component({
  selector: 'app-food-pizzas',
  providers: [
    // FoodService
    {
      provide: FoodService,
      useFactory: (http) => new FoodService(http, '/api/pizzas'),
      deps: [HttpClient]
    }
  ],
  ...
})
```
<br>

`app/food/components/food-drinks.component.ts`
``` ts
@Component({
  selector: 'app-food-drinks',
  providers: [
    // FoodService
    // { provide: FoodService, useClass: FoodMockService }
    {
      provide: FoodService,
      useFactory: (http) => new FoodService(http, '/api/drinks'),
      deps: [HttpClient]
    }
  ],
  ...
})
```
<br>

# 6.5 Providers and useExisting

## 6.5.1 Refactoriing
---

<br>

> `FoodService`

Vamos a poner en el servicio `FoodService` un método para cada tipo de producto.

`app/foods/food.service.ts`
``` ts
...
@Injectable()
export class FoodService {
  constructor(
    private http: HttpClient,
    // private api: string
    @Inject(FOOD_API_TOKEN) private api: string
  ) {
    console.log('FoodService::constructor()');
  }

  getPizzas(): Observable<Food[]> {
    return this.http.get<Food[]>(this.api + `/pizzas`);
  }

  getDrinks(): Observable<Food[]> {
    return this.http.get<Food[]>(this.api + '/drinks');
  }

  getSides(): Observable<Food[]> {
    return this.http.get<Food[]>(this.api + '/sides');
  }

}
```
<br>

> `FoodModule`

En el módulo `FoodModule` inyectamos el token con el Api.

`app/food/food.module.ts`
``` ts
  providers: [
    { provide: FOOD_API_TOKEN, useValue: '/api'}
  ],
```
<br>

> Componentes `food-*.component.ts`

Y en cada componente usamos su correspondiente método para obtener los datos.

`app/food/components/food-pizzas.component.ts`
``` ts
  ...
  providers: [
    FoodService
  ],
  ...

  ngOnInit() {
    this.items$ = this.foodService.getPizzas();
  }
}
```
<br>

`app/food/components/food-drinks.component.ts`
``` ts
  ...
  providers: [
    FoodService
  ],
  ...

  ngOnInit() {
    this.items$ = this.foodService.getDrinks();
  }
}
```
<br>

`app/food/components/food-sides.component.ts`
``` ts
  ...
  providers: [
    FoodService
  ],
  ...

  ngOnInit() {
    this.items$ = this.foodService.getSides();
  }
}
```
<br>

> Nota

En el componente `FoodPizzasComponent` si tecleamos `this.foodService.` la ayuda contextual nos sugiere los tres métodos (getDrinks, getPizzas y getSides) pero en este componente sólo tiene sentido uno.

<br>

## 6.5.2 Crear clase abstracta
---

Vamos a crear una clase abstracta `PizzaService` con un solo método `getPizzas` , pero le diremos que use la instancia existente de FoodService.

<br>

`app/food/components/food-pizza.component.ts`
```ts
export abstract class PizzaService {
  getPizzas: () => Observable<Food[]>
}

@Component...
```
<br>

Definimos un provider `PizzaService`, pero le decimos que use una instancia del servicio `FoodService`

`app/food/components/food-pizza.component.ts`
```ts
  providers: [
    FoodService,
    { provide: PizzaService, useExisting: FoodService }
  ],
```
- Necesitamos registrar también `FoodService`

<br>

Y ahora en el constructor inyectamos un servicio de tipo `PizzaService`

`app/food/components/food-pizza.component.ts`
```ts
  constructor(private foodService: PizzaService) { }
```
<br>

> Notas

- `abstract class PizzaService` crea un interfaz público y esconde el resto de métodos.

- `{ provide: PizzaService, useExisting: FoodService }` indica que cuando usemos `PizzaService`, la inyección de dependencias le pase una instancia de `FoodService`

- Definimos el servicio como `foodService: PizzaService` para que Typescript reconozca que metodos tiene

Si ahora escribimos `this.foodService.` ya solo nos sugiere un método (getDrinks)

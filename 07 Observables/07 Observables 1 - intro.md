# 7.1 Observables. Introducción

https://rxjs.dev/

https://www.learnrxjs.io/


# 7.1.1 Create observable

```ts
import { Observable } from "rxjs";

const observable: Observable<number> = Observable.create(observer => {
  let value = 0;
  const interval = setInterval(() => {
    observer.next(value);
    value++;
  }, 1000);

  return () => clearInterval(interval);
})

const subscription = observable.subscribe(
  value => console.log(value),
  (err) => console.error(err),
  () => console.log('Completed!')
);

setTimeout(() => subscription.unsubscribe(), 10000);
```

- El observable crea un productor que emite datos hasta que se completa o genrea un error.

- Para consumir esos datos es necesario suscribirse utilizando un `observer`, que es un callback con tres parámetros (next, error, complete)

- El consumidor puede cancelar la suscripción al observable

<br>

# 7.1.2 Cancelar suscripciones

Si un observable no se completa, y no cancelamos la subscipción se queda en memoria.
Todas las suscripciones a observables que no se cancelen automáticamente, tenemos que cancelarlas.

Cuando un componente se destruye, no significa que cancele las suscripciones automáticamente.

<br>

# 7.1.3 Subject

Es un tipo de observable que permite que la emisión de valores se comparta con muchos observadores.

Similar al EventEmitter.

Es a la vez un Observable y un Observer.
- Como observable nos podemos suscribir a el
- Como observer puede emitir valores (invocando `next()`)

Para prevenir que se puedan emitir valores existe el método `subject.asObservable()`

Tipos de Subject

| Operador | Descripción |
| ---         | --- |
| Subject     | Estandar, sin valor inicial ni replay |
| AsyncSubject | Emite todos los valores cuando se completa |
| BehaviorSubject | Cuando te suscribes emite el ultimo valor emitido o un valor por defecto,<br> y continua emitiendo valores  |
| ReplaySubject | Emite todos los valores emitidos previamente. <br> Se puede definir el tamaño de buffer o la duración de tiempo |
<br>

<br>

# 7.1.4 Cold vs Hot Observables

| Cold | Hot |
| ---         | --- |
| por defecto |     |
| **unicast** : una emisión por suscriptor | **multicast**: se comparte la emisión |
| Emisión comienza despues del `subscribe` | Emisión empieza sin `susbscriptions` |
<br>


## 7.1.5 Operators
---

<br>

> Transformation Operators
---

| Operador | Descripción |
| --- | --- |
|  map() | Turn an array, promise or iterable into an observable. |
|  pluck() | Selecciona una o más propiedades para emitir |
|  concatMap() | `Map` a inner observable, se suscribe y emite en orden. <br>Espera a que se complete uno antes de mezlar los siguientes |
|  switchMap() | `Map` a un observable, completa anterior, y emite valores |

<br>

> Filtering Operators
---

| Operador | Descripción |
| --- | --- |
| filter() | Emite solo los valores que pasan una condición |
| debounceTime(n) | Descarta un valor si ha pasado poco tiempo desde el anterior |
| distintUntilChanged() | Emite un valor si es distinto del anterior |
| take(n) | Emite `n` valores y se completa |
| takeUntil(obs) | Emite valores hasta que el observable `obs` emita un valor |

<br>

> Combination Operators
---

| Operador | Descripción |
| --- | --- |
| combineLatest(s1, s2) | Cada vez que un observable emite un valor, <br>se emite el último valor emitido por cada uno de ellos |
| concat(s1, s2) | Mezcla observables en orden secuencial. <br>Hasta que uno no se completa, no se suscribe al siguiente |
| merge(s1, s2) | Mezcla multiples observables |
| forkJoin([s1, s2]) | Cuando todos han completado, se emite el último valor de cada uno. <br>Si uno da error, genera error y se pierden todos |
| pairwise() | Emite el valor anterior y actual en un array |
| startWith() | Emite valores iniciales |

<br>

> Error Handling
---

| Operador | Descripción |
| --- | --- |
| catchError() | Captura errores y emiteun valor |
| retry() | Reintenta un observable n veces si ocurre un error |
| throwError() | Provoca un error |

<br>

> Creation Operators
---

| Operador | Descripción |
| --- | --- |
| of(a, b) | Emite una lista de valores |
| from() | Covierte un array, promise o iterable en un observable |
| fromEvent() | Convierte un evento en un observable |
| EMPTY | Observable que completa sin emitir |
| interval | Emite valores en secuencia cada x milisegundos |
| timer(1000, 5000) | Emite valores en secuencia: <br>Primero un 0 en 1s, luego sigue cada 5s |
| Angular HttpClient | Peticiones HTTP |

<br>

> Multicast Operators
---

Convierten un observable en multicast.
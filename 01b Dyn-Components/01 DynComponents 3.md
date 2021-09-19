# Advanced Components

https://github.com/ultimatecourses/angular-pro-src


- `app/demo/onpush/demo-onpush.component.ts`
- `app/demo/onpush/example-one.component.ts`
- `app/demo/onpush/example-two.component.ts`

<br>

Activamos la demo en el `AppComponent`

`app/app.component.ts`
```html
    <h1>{{ title }}</h1>
    <app-demo-onpush></app-demo-onpush>
```
<br>

# Código inicial


# Paso 21. `ChangeDetectionStrategy.OnPush` and Inmutability

Inicialmente los dos componentes tienen la estrategia por defecto
- changeDetection: ChangeDetectionStrategy.Default

<br>

Al definir un componente podemos definir la estrategia de ChangeDetection
- ejemplo 1 => ChangeDetecion.OnPush
- ejemplo 2 =>  ChangeDetection.Default

<br>

> Ponemos en `ExampleOneComponent` la estrategia `OnPush`
> - changeDetection: ChangeDetectionStrategy.Onpush

`app/demo/onpush/example-one.component.ts`
```ts
@Component({
  selector: 'app-example-one',
  changeDetection: ChangeDetectionStrategy.OnPush,
  ...
})
```
<br>

> Ejecutar

Al añadir una propiedad nueva al objeto user que no existe

Si modificamos una propiedad del objeto que ya existe

Si modificamos el objeto user, creando un nuevo objeto con una referencia nueva

Angular es más rapido trabajando con objeto inmutables porque en cada changeDetector no tiene
que comprobar cada propiedad del objeto, solo si el objeto tiene la misma referencia.


Angular es mas rápido porque no tiene que hacer tantos cálculos para detectar si debe volver a renderizar la plantilla.

Este ejemplo sólo mira, como angular comprueba el input de distinta forma en default y en onPush.
Pero no mira que en onPush solo depende de cambios en inputs, no en otros cambios de estado.
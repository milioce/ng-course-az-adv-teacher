# 9.4 Injection Tokens poco pesados.

https://angular.io/guide/lightweight-injection-tokens

Angular cuando compila intenta reducir el tamaño del bundle.

Si un componente o servicio no es usado nunca, el compilador puede eliminar su codigo del bundle generado. Este proceso se llama `tree-shaking`

Esto es especialmente importante en librerías, para aseguranos que si la apliación usa solo una parte, el código de los objetos no utlizados sea eliminado del bundle generado al compilar.

 Todos los servicios registrados como provider en el root injector, son `tree-shakable`. Angular puede detectar facilmente si no los usamos y no incluirlos en el compilado.

El problema son los componentes que inyectamos en nuestra aplicación

<br>

# 4.1 Token retained

Imagenemos un componente <lib-card> que admite como componentes hijos `lib-header` o `lib-body`.

```ts
<lib-card>
  <lib-header>...</lib-header>
</lib-card>
```
- El componente `lib-header` es opcional, podemos no utilizarlo en nuestra aplicación.

<br>

Una implementación habitual del componente seria:

```ts
class LibCardComponent {
  @ContentChild(LibHeaderComponent) header: LibHeaderComponent|null = null;
}
```
<br>

- `@ContentChild(LibHeaderComponent)` es una **referencia como valor**
- `header: LibHeaderComponent` es una **referencia como tipo**

El compilador elimina las referencias como tipo después de la conversión desde TS a JS, así que no impacta en el `tree-shaking`

El compilador <u>retiene</u> las referencias como valor, lo que impide que se elimine el código.

- Los tokens que se usan como especificadores de tipo, se eliminan al convertirse a JS.

- Los token que se usan para inyectar dependencias son necesarios en tiempo de ejecución y no se eliminan.

<br>

``` ts
class MyComponent {
  constructor(@Optional() other: OtherComponent) {}

  @ContentChild(OtherComponent) other: OtherComponent|null;
}
```
- `OtherComponent` inyecta el componente en el constructor, se retiene
- ` @ContentChild(OtherComponent)` usa el componente como valor, se retiene

# 4.1.2 Injection Token livianos

La solución es usar un `injectionToken` poco pesados.

```ts
abstract class LibHeaderToken {}

@Component({
  selector: 'lib-header',
  providers: [
    {provide: LibHeaderToken, useExisting: LibHeaderComponent}
  ]
  ...,
})
class LibHeaderComponent extends LibHeaderToken {}

@Component({
  selector: 'lib-card',
  ...,
})
class LibCardComponent {
  @ContentChild(LibHeaderToken) header: LibHeaderToken|null = null;
}
```
<br>

- Ahora `LibCardComponent` ya no tiene ninguna referencia a `LibHeaderComponent`, eso permite que se pueda eliminar el código de `LibHeaderComponent`

- El `LibHeaderToken` se conserva pero es una clase abstracta sin implementación, lo que no afecta al tamaño del bundle.

La buena práctica es usar siempre `lightweight injection token` para la definición del API de una ibrería.

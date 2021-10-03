# 1. Buenas prácticas en Angular

https://angular.io/guide/security

```bash
$ ng generate application bestPractices -s -t --style=scss --routing=true
```
<br>

# 1 Rendimiento / Consistencia

# 1.1 Change detection Strategy OnPush

Define como es la detección de cambios.

- La detección de cambios trata de detectar qué cambios que se han producido en el modelo, para propagarlos.

- En el árbol de componentes (de arriba a abajo) mira si el modelo del que depende ha cambiado, y la vista del componente (Dom)

<br>

Hay dos estrategias de detección de cambios:
- Default
- OnPush

## 1.1.1 ChangeDetectionStrategy.Default
---

Angular no sabe de qué depende el componente, asi que lanzará una comprobación cada vez que algo cambie.

En concreto comprobará para cada evento del navegador, timers, XHR, Promises.

<br>

## 1.1.2 ChangeDetectionStrategy.OnPush
---

Con esta estrategia le informamos a Angular que el componente depende de cambios en los Inputs, eventos activados en el componente o sus hijos.
Tambien podemos lanzar la detección de cambios explicitamente con el metodo componentRef.markForCheck()

<br>

> Ver en el proyecto `dyn-componentes` la demo `app-demo-onpush`

<br>

# 1.2 TrackBy en *ngFor

Cuando renderizamos una lista de elementos con `ngFor` y <u>añadimos o eliminamos elementos dinámicamente</u> de esa lista, angular no sabe que elemento ha cambiado.

Saca la lista completa del DOM y vuelve a insertarla.

Una forma de optimizar ese proceso en listas largas es usar el `trackBy` para informarle a Angular de un identificador para cada elemento de la lista.

  ```ts
  <li *ngFor="let item of items; trackBy: trackByItems">
    {{ item.id }} {{ item.name }}
  </li>

  trackByItems(index, item) {
    return item.id;
  }
  ```

En listas estáticas, que no van a cambiar una vez se han mostrado, no hace falta usarlo.

<br>

# 1.3 Pipes puras

Una función pura es aquella que dada una misma entrada, devuelve siempre la misma salida y no tiene efectos secundarios.

Una `Pipe` pura es aquella que aplicada sobre un mismo valor, devuelve siempre la misma salida, y no tiene efectos secundarios.

Angular por defecto define las `Pipes`como puras, y solo volverá a ejecutarlas si cambia el valor del `Input`.

Si el `Input` es un tipo de dato primitivo (string, number, boolean) comprobará si cambia su valor. En `Date`, `Array`, `Function`, `Object` si cambia su referencia.

Es decir, hay que usar datos inmutables.

<br>

# 1.4 `Lazy loading` en módulos

Si la aplicación tiene muchos módulo debemos crear los módulo como `lazy-loading` para reducir los módulos que carga en la página principal.

Dependiendo de la cantidad de módulos que tengamos, del peso de cada uno,  y del uso que hagamos de ellos, podemos definir diferentes estrategias de precarga de módulos.

<br>

# 1.5 Cancelar suscripciones de observables

- Cancelar suscriciones que no se completen automáticamente.

- Usar async pipe.
  Cuando el componente que usa el async en su plantilla se destruye, se cancela la suscripción automáticamente.

<br>

# 1.6 Utilizar componentes reutilizables

Dividir un componente en piezas más pequeñas, creando componentes reutilizalbles.

# 1.7 Estructura de directorios coherente

Utilizar una estructura de directorios coherente, que nos facilite identificar donde tenemos los diferentes objetos de la aplicación.

Por ejemplo,

https://www.ideas2it.com/blogs/angular-development-best-practices/

<br>

# 1.8 Estandar de codificación

Seguir un estandar de codificación y herramientas para que el editor nos ayude a hacerlo.

El objetivo es conseguir un código mas limpio y consistente.

Podemos usar la guía de estilo de Angular, si el proyecto no define ninguna.
  - 400 lineas de código por fichero
  - 75 lineas de código por función o método
  - Nomenclatura de ficheros
  - etc

Usar reglas lint para typescript y SCSS.

https://angular.io/guide/styleguide

<br>

# 1.9 Simplificar imports

- Usar un fichero `index.ts`
  - Se mantienen todos los objetos relacionados juntos
  - Simplifica los imports, importando directamente la carpeta

- Usar `compilarOption/paths` en el fichero `tsconfig.json`

```json
{
    "compilerOptions": {
        ...
        "paths": {
            "@app/core/*": [ "src/app/core/*" ],
            "@app/shared/*": [ "src/app/shared/*" ],
            "@app/shared-components": [ "src/app/shared/components/index" ],
            "@environment/*": [ "src/environments/*" ],
        }
    }
}
```
<br>

```ts
import { MyComponent } from '../../../../shared/components/my-component';
import { MyComponent } from '@app/shared/componentes/my-component';
import { MyComponent } from '@app/shared-components'; // index.ts
```
<br>

# 1.10 Usar `property binding`

Usar property binding siempre que sea posible para establecer propiedades de los elementos.

- `<img [src]="itemImageUrl">`

o Attribute binding

- `<p [attr.atributo]="expresion">`
  <br> `[attr.aria-label]="expresion"`
  <br> `aria-label="literal"`

En property bindings, las expresiones no deben tener efectos secundarios.No deben modificar variables.

<br>

# 1.11 Cachear respuestas del API

Cachear respuestas del API cuando los datos recibidos no cambien con frecuencia.

<br>

# 1.12 Usar CDK Virtual Scroll para listas largas

Angular Material incluye componentes para mostrar listas con gran cantidad de elementos.

Se basa en informar de la altura de un elemento y el componente ajusta la altura del contenedor a la altura de los elementos que contiene, y renderizando solo los elementos que se ven.

https://material.angular.io/cdk/scrolling/overview#virtual-scrolling

```ts
<ul>
  <cdk-virtual-scroll-viewport itemSize="100">
    <ng-container *cdkVirtualFor="let item of items">
   	  <li> {{item}}</li>
    </ng-container>
  </cdk-virtual-scroll-viewport>
</ul>
```
<br>

# 1.13 Usar environment variables.

Angular perimite declarar variables de configuración por entornos.

Por defecto crea dos entornos: development y production, pero podemos crear más entornos.

<br>

# 1.14 Usar AOT en producción

En las últimas version Angular ya compila con AOT en modo production.

Además de obtener mejor rendimiento, previene XSS injection porque precompila las plantillas.

No permite crear plantillas dinámicamente concatenando datos de usuariono confiables.

<br>
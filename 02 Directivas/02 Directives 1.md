# Directives

22. Creating a custom attribute directive
23. @HostListener and host Object
24. Understanding @Hostbinding
25. Use the exportAs property with template refs
26. Creating a custom structural Directive

# Paso 0. Initial files

 - `app/demo/demo-credit-card.component.ts`
 - `app/demo/demo-my-for.component.ts`
 - `app/shared/credit-card.directive.ts`
 - `app/shared/tooltip.directive.ts`
 - `app/shared/my-for.directive.ts`
 - `app/shared/shared.module.ts`
 - `app/app.component.ts`
 - `app/app.module.ts`
 - `src/styles.scss`

<br>

> En el `AppComponent` activamos la demo `<demo-credit-card></demo-credit-card>`

`app/app.component.ts`
```html
  template: `
    <demo-credit-card></demo-credit-card>
  `,
 ```
<br>

# Paso 1. Creating a custom attribute directive

En la página `demo-credit-card.component.ts` tenemos un formulario.

`app/demo/demo-credit-card.component.ts`
``` html
    <h1>Demo Credit Card Directive</h1>
    <div>
      <label>
        Credit Card Number
        <input
          name="credit-card"
          type="text"
          placeholder="Enter your 16-digit card number">
      </label>
    </div>
```
<br>

> Vamos a crear una directiva para formatear el número de una tarjeta   bancaria, añadiendo especios cada cuatro digitos.

<br>

## 1.1 Crear la directiva

En el fichero `app/shared/credit-card.directive.ts` tenemos el código de una directiva.

<br>

`app/shared/credit-card.directive.ts`
``` ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[credit-card]'
})
export class CreditCardDirective {
  constructor(private element: ElementRef) {
    console.log(this.element);
  }
}
```


- Definimos una clase con el decorador `@Directive()`, de forma parecida a como creamos un componente.
- Definimos un selector `[credit-card]`
- En el constructor le inyectamos `ElementRef`, el elemento sobre el que se aplica la directiva.
- Está declarada en el módulo `SharedModule`

<br>

> Añadimos la directiva al html

`app/demo/demo-credit-card.component.ts`
```html
  <input
    name="credit-card"
    type="text"
    placeholder="Enter your 16-digit card number"
    credit-card>
```
<br>

## 1.2 @HostListener and host Object

La directiva `@HostListener()` nos permite escuchar un evento del `host`, que es el que tiene la directiva.

- El primer parámetro es el evento que queremos escuchar: el evento `input`
- El segundo parámetro es un array con los argumentos que queremos pasar al manejador cuando ocurra.
- y definimos el manejador que se ejecutará cuando ocurra el evento

`app/shared/credit-card.directive.ts`
``` ts
export class CreditCardDirective {
  @HostListener('input', ['$event'])
  onKeyDown(event: KeyboardEvent) {
    console.log(event);
  }
  ...
}

```

Como argumentos podemos pasarle
- `$event`: El evento nativo, en este caso KeyboardEvent
- `$event.target`: El elemento que recibe el evento

<br>

Definimos el comportamiento de la directiva
- Eliminamos espacios en blanco
- Ignoramos más de 16 caracteres
- Insertamos un espacio en blanco cada cuatro caracteres

`app/shared/credit-card.directive.ts`
``` ts
export class CreditCardDirective {
  @HostListener('input', ['$event'])
  onKeyDown(event: KeyboardEvent) {
    const input = event.target as HTMLInputElement;

    // Eliminar espacios en blanco
    let trimmed = input.value.replace(/\s+/g, '');

    // Ignorar más de 16 espacios
    if (trimmed.length > 16) {
      trimmed = trimmed.substr(0, 16);
    }

    // Insertar un espacio cada 4 caracteres
    const numbers = [];
    for (let i = 0; i < trimmed.length; i += 4) {
      numbers.push(trimmed.substr(i, 4));
    }
    input.value = numbers.join(' ');

  }

  constructor(private element: ElementRef) {
    console.log(this.element);
  }
}
```
<br>

## 1.3 Understanding @Hostbinding

`@Hostbinding()` sirve para hacer binding entre una propiedad del host y una variable del ts.

- Vamos a conectar `style.border` con la variable `this.border`.

`app/shared/credit-card.directive.ts`
``` ts
export class CreditCardDirective {
  @HostBinding('style.border') border: string;

  ...

  input.value = numbers.join(' ');

  // Si el valor introducido contiene un no digito, mostramos un border rojo
  if (/[^\d]+/.test(trimmed)) {
    this.border = '1px solid red';
  } else {
    this.border = '';
  }
}
```
<br>

# Paso 2. Use the exportAs property with template refs

`exportAs` sirve para acceder a la Directiva desde una referencia de plantilla.

## 2.1 Directiva tooltip

<br>

> Partimos del esqueleto de una directiva

`app/shared/tooltip.directive.ts`
``` ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[tooltip]',
  exportAs: 'tooltip'
})
export class TooltipDirective {
  constructor(private element: ElementRef) { }
}
```
<br>

> Queremos crear una nueva directiva tooltip
> - Inserta el tooltip en el DOM
> - Ofrece dos métodos públicos `show` y `hide` para mostrar y ocultar el tooltip.

`app/shared/tooltip.directive.ts`
``` ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[tooltip]',
  exportAs: 'tooltip'
})
export class TooltipDirective {
  constructor(private element: ElementRef) { }

  show() {}

  hide() {}
}

```
<br>

> Añadimos al HTML un label con el tooltip que deseamos

`app/demo/demo-credit-card.component.ts`
```html
      <label tooltip="3 digits, back of your card" #mytooltip="tooltip">
        Enter your security code
        <span
          (mouseover)="mytooltip.show()"
          (mouveout)="mytooltip.hide()">
          (?)
        </span>
        <input type="text">
      </label>
```
<br>

> Creamos el nuevo elemento `div` que contendrá el tooltip

`app/shared/tooltip.directive.ts`
``` ts
export class TooltipDirective {
  tooltipElement = document.createElement('div');
  visible = false;

  constructor(private element: ElementRef) { }
  ...
}
```
<br>

> `tooltip` es el nombre de la directiva y también un input para asignarle un valor.
> - Vamos a utilizar una propiedad con un `setter` para usar el valor

`app/shared/tooltip.directive.ts`
``` ts
export class TooltipDirective {
  tooltipElement = document.createElement('div');
  visible = false;

  @Input() set tooltip(value) {
    this.tooltipElement.textContent = value;
  }

  constructor(private element: ElementRef) { }
  ...
```
<br>


> Cuando se inicialice la directiva (`onInit`) insertamos el tooltip en el Dom

`app/shared/tooltip.directive.ts`
``` ts
export class TooltipDirective implements OnInit {
  ...
  constructor(private element: ElementRef) { }

  ngOnInit() {
    this.tooltipElement.className = 'tooltip';
    this.element.nativeElement.appendChild(this.tooltipElement);
    this.element.nativeElement.classList.add('tooltip-container');
  }

  show() {}

  hide() {}
}

```
<br>

- Con el inspector podemos ver
  - como se ha insertado el elemento `div.tooltip` en el Dom
  - como en el `<label>` se ha añadido la case `tooltip-container`

<br>

> Ahora implementamos los métodos publicos `show` y `hide`

`app/shared/tooltip.directive.ts`
``` ts
  show() {
    this.tooltipElement.classList.add('tooltip--active');
  }

  hide() {
    this.tooltipElement.classList.remove('tooltip--active');
  }
```
<br>

> El código final de la directiva quedaría así

`app/shared/tooltip.directive.ts`
``` ts
import { Directive, ElementRef, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[tooltip]',
  exportAs: 'tooltip'
})
export class TooltipDirective implements OnInit {
  tooltipElement = document.createElement('div');
  visible = false;

  @Input() set tooltip(value) {
    this.tooltipElement.textContent = value;
  }

  constructor(private element: ElementRef) { }

  ngOnInit() {
    this.tooltipElement.className = 'tooltip';
    this.element.nativeElement.appendChild(this.tooltipElement);
    this.element.nativeElement.classList.add('tooltip-container');
  }

  show() {
    this.tooltipElement.classList.add('tooltip--active');
  }

  hide() {
    this.tooltipElement.classList.remove('tooltip--active');
  }
}
```
<br>


# Paso 3. Creating a custom structural Directive

Vamos a crear una nueva directiva `myFor` que haya lo mismo que el `ngFor`

## 3.1 Activar la demo `<my-for>`

`app/app.component.ts`
```html
  template: `
    <demo-myfor></demo-myfor>
  `,
```
<br>

## 3.2 Analizamos la directiva `ngFor`

`app/demo/demo-my-for.component.ts`
```html
  template: `
    <h1>Demo Custom Structural Directive</h1>
    <div>
      <ul>
        <li *ngFor="let item of items; let i = index">
          {{i}} - {{ item.name }} - {{ item.location }}
        </li>
      </ul>
    </div>
  `
```
<br>

Duplicamos el `<li>` con el equivalente en `ng-template`

`app/demo/demo-my-for.component.ts`
```html
      <ul>
        <li *ngFor="let item of items; let i = index">
          {{i}} - {{ item.name }} - {{ item.location }}
        </li>
        <ng-template ngFor [ngForOf]="items" let-item let-i="index">
          <li>
            {{i}} - {{ item.name }} - {{ item.location }}
          </li>
        </ng-template>
      </ul>
```
- `ngFor` es la directiva
- `ngForOf` es un input para pasarle los datos
- `let-item` es el `$implicit` que contendrá el valor de cada iteración
- `let-i="index"` es otra variable del context (i dentro del template, index en el contexto)

<br>

## 3.3 Crear la directiva `myFor`

Vamos a crear una nueva directiva `myFor`.
- Modificamos los dos `<li>` por la nueva directiva

`app/demo/demo-my-for.component.ts`
```html
      <ul>
        <li *myFor="let item of items; let i = index">
          {{i}} - {{ item.name }} - {{ item.location }}
        </li>
        <ng-template myFor [myForOf]="items" let-item let-i="index">
          <li>
            {{i}} - {{ item.name }} - {{ item.location }}
          </li>
        </ng-template>
      </ul>
```
<br>

> Creamos la directiva `app/shared/my-for.directive.ts`

- Creamos como selector `'[myFor][myForOf]'`
- Creamos un `@Input() set myForOf`

`app/shared/my-for.directive.ts`
``` ts
import { Directive, Input } from "@angular/core";

@Directive({
  selector: '[myFor][myForOf]'
})
export class MyForDirective {
  @Input()
  set myForOf(collection) {
    console.log(collection);
  }
}
```
- En la consola vemos los items

<br>


> Ahora creamos el constructor
> - Inyectamos el `view: ViewContainerRef`
> - Inyectamos el `template: TemplateRef<any>`

`app/shared/my-for.directive.ts`
``` ts
  constructor(
    private view: ViewContainerRef,
    private template: TemplateRef<any>
  ) {
  }
```
<br>

> Para cada elemento del array de items: creamos una vista a partir de la plantilla y la insertamos en el contenedor.
> - `$implicit` es cada elemento item
> - `index` es el indice del array de items

`app/shared/my-for.directive.ts`
``` ts
  set myForOf(collection) {
    collection.forEach((item, index) => {
      this.view.createEmbeddedView(this.template, {
        $implicit: item,
        index: index
      });
    });
  }

```
<br>

## 3.4 Añadimos elementos dinámicamente

Añadirmos en el constructor del `DemoMyForComponent` un item más a los dos segundos.

`app/demo/demo-my-for.component.ts`
``` ts
  constructor() {
    setTimeout(() => {
        this.items = [
          ...this.items,
          { name: 'Emilio', location: 'Zaragoza' }
        ];
    }, 2000);
  }
```
<br>

- Vemos que en el Angular detecta que cambia el input y vuelve a pintar toda la lista de nuevo, debajo de la que ya existe.

- Tenemos que limpiar el viewContainer antes de pintar los items.

<br>

`app/shared/my-for.directive.ts`
``` ts
  set myForOf(collection) {
+   this.view.clear();
    collection.forEach((item, index) => {
      this.view.createEmbeddedView(this.template, {
        $implicit: item,
        index: index
      });
    });
  }
```
<br>

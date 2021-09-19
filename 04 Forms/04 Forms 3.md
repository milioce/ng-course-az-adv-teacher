# Reactive Forms

# 0. Initial files

Vamos a crear un control custom para incrementar y decrementar la cantidad.

Partimos del código del componente `CounterComponent`
- `app/cart/components/counter/counter.component.ts`

Y podemos verlo si incluimos el selector.

```html
          <input type="number"
            step="1"
            min="1"
            max="100"
            formControlName="quantity">

+           <app-counter></app-counter>
```
<br>

# 1. Custom form control base

Ahora nos falta convertir este componente en un control custom de formulario, que permita usar el formControl o formControlName.

- `ControlValueAccesor` nos permite que nuestro componente se integre con Angular Forms API. (Se habla con ReactiveForm)
- Necesitamos registrar nuestro componente, extendiendo el `NG_VALUE_ACCESSOR` (token que provee el ControlValueAccesor a un form control).

<br>

> Paso 1. Definimos un provider
> - Extiende el `NG_VALUE_ACCESSOR`
> - forwardRef() para usar una referencia a una clase antes de ser instanciada.

`app/cart/counter/counter.component.ts`
```ts
const COUNTER_VALUE_ACCESOR: Provider = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => CounterComponent),
  multi: true
}

@Component({
  selector: 'app-counter',
+ providers: [ COUNTER_VALUE_ACCESOR ],
  styleUrls: ['counter.component.scss'],
  template: `...`
})
```
<br>

> Paso 2: Implementar el interface `ControlValueAccessor`
 - `writeValue`:
   - Forms API lo usará cuando cambie el valor del control.
   - Un cambio en el modelo se propaga a la vista

 - `registerOnTouched`, `registerOnChange`:

   - Angular Forms API invoca estos métodos durante la inicializacion para registrar los callbacks que actualizan el modelo.
   - Un cambio en la vista se propaga al modelo (invocando los callbacks.
   - Nosotros los cogemos y los guardamos en una propiedad privada para usarlos.

<br>

`app/cart/counter/counter.component.ts`
```ts
export class CounterComponent implements OnInit, ControlValueAccessor {
  @Input() step: number = 10;
  @Input() min: number = 10;
  @Input() max: number = 1000;

  value: number = 10;
+ private onModelChange: Function;
+ private onTouch: () => Function;

  constructor() { }

  ngOnInit() { }

+ registerOnChange(fn) {
+   this.onModelChange = fn;
+ }

+ registerOnTouched(fn) {
+   this.onTouch = fn;
+ }

+ writeValue(value) {
+   this.value = value || 0;
+ }

  ...
}
```
<br>

> Paso 3. Invocar los callbacks.

`app/cart/counter/counter.component.ts`
```ts
export class CounterComponent implements OnInit, ControlValueAccessor {
  ...

  increment() {
    if (this.value < this.max) {
      this.value = this.value + this.step;

      // Necesitamos invocar el callback registrado por `registerOnChange
      // cuando nuestro componente cambie.
      this.onModelChange(this.value);
    }

    // Necesitamos invocar el callback registrado por `registerOnTouched`
    // cuando el usuario interactue con nuestro componente.
    this.onTouch();
  }

  decrement() {
    if (this.value > this.min) {
      this.value = this.value - this.step;
      this.onModelChange(this.value);
    }
    this.onTouch();
  }
}
```
<br>

> Paso 4. Usar el componente con formControlName

`app/cart/items/cart-items.component.ts`
```html
            <app-counter
              [step]="1"
              [min]="1"
              [max]="100"
              formControlName="quantity">
            </app-counter>
```
<br>

> Paso 5. Adding keyboard events to out control

Vamos a añadir la posibilidad de usar nuestro componente con el teclado.

<br>

> Paso1. Capturamos los eventos `keydown`, `blur` y `focus`

- Capturamos el evento `keydown` para controlar el movimiento con teclado.
- Capturamos eventos `blur` y `focus` para resaltar el componente.
- Necesitamos añadir `tabindex` para acceder con teclado al `div`


`app/cart/counter/counter.component.ts`
```html
    <div class="stock-counter">
      <div>
+       <div tabindex="0"
+         (keydown)="onKeyDown($event)"
+         (blur)="onBlur($event)"
+         (focus)="onFocus($event)">
          <p>{{ value }}</p>
          <div>
            <button type="button"
              (click)="increment()"
              [disabled]="value === max">+</button>
            <button type="button"
              (click)="decrement()"
              [disabled]="value === min">-</button>
          </div>
        </div>
      </div>
    </div>
```
<br>


`app/cart/counter/counter.component.ts`
```ts
export class CounterComponent implements OnInit, ControlValueAccessor {
  ...

  onKeyDown(event: KeyboardEvent) {}

  onBlur(event: FocusEvent) {}

  onFocus(event: FocusEvent) {}
}
```
<br>

> Paso 2. Implementamos el `onKeyDown()`

`app/cart/counter/counter.component.ts`
```ts
  onKeyDown(event: KeyboardEvent) {
    const handlers = {
      ArrowDown: () => this.decrement(),
      ArrowUp: () => this.increment()
    };

    if (handlers[event.key]) {
      handlers[event.key]();
      event.preventDefault();
      event.stopPropagation();
    }

    this.onTouch();
  }
```
<br>

# 2. Validators object for FormControls

Vamos a añadir dos campos más al formulario: Código postal y Cupón de descuento.

## 2.1 Crear nuevos campos en el formulario
<br>

> Paso 1. Añadimos los campos a la definición del formulario

`app/cart/cart.component.ts`
```ts
  private buildForm() {
    this.form = this.fb.group({
+     store: this.fb.group({
+       postalCode: '',
+       coupon: ''
+     }),
      selector: this.fb.group({
        product_id: '',
        quantity: 1
      }),
      cart: this.fb.array([])
    });
  }
```
<br>

> Paso 2. Tenemos el componente `CartCouponComponent` creado en la carpeta `app/cart/components/cart-coupon.component.ts`.

Ya tenemos registrado el componente en `CartModule`

Nos falta solo incluir el componente en el formulario

<br>

> Paso 3. Incluir el componente en `CartComponent`, el formulario padre.

`app/cart/cart.component.ts`
```html
      <form [formGroup]="form">

        <app-cart-coupon
          [parent]="form">
        </app-cart-coupon>

        ...
      </form>
```
<br>

## 2.2 Añadimos Validaciones a los controles `postalCode` y `coupon`

<br>

> Añadimos validaciones en la definición del formulario.

`app/cart/cart.component.ts`
```ts
  private buildForm() {
    this.form = this.fb.group({
      store: this.fb.group({
+       postalCode: ['', Validators.required],
+       coupon: ['']
      }),
      selector: this.fb.group({
        product_id: '',
        quantity: 1
      }),
      cart: this.fb.array([])
    });
  }
```

- El segundo elemento del array son las validaciones (síncronas)
- El botón `Submit` está disabled hasta que el formulario sea válido.

<br>

> Añadimos un mensaje de error

`app/cart/cart.component.ts`
```ts
  template: `
    <input
      type="text"
      placeholder="Postal code"
      formControlName="postalCode">
    <div class="error"
      *ngIf="postalCode.hasError('required') && postalCode.touched">
      Postal code is required
    </div>
  `
  ...

  get postalCode() {
    return this.parent.get('store.postalCode') as FormControl;
  }

```
<br>

> Simplificamos el código

```ts
  template: `
    <input
      type="text"
      placeholder="Postal code"
      formControlName="postalCode">
    <div class="error"
      *ngIf="postalCode.hasError('required') && postalCode.touched">
      Postal code is required
    </div>
  `
  ...

  errorRequired(name: string): boolean {
    const control = this.parent.get(`store.${name}`);

    return control.hasError('required') && control.touched;
  }

```
<br>

# 2.3 FormControl custom validators

> Creamos un nuevo fichero `cart.validators.ts` para definir validaciones.

`app/cart/validators/cart.validators.ts`
```ts
import { AbstractControl } from "@angular/forms";

export class CartValidators {
  static validCoupon(control: AbstractControl) {
    const regexp = /^[a-z]\d{3}$/i;
    const valid = regexp.test(control.value);

    return valid ? null : { couponInvalid: true };
  }
}
```
- Devuelve null si el control es válido
- Si hay error se devuelve un objeto
  - key: codigo de error
  - value: true o un objeto con datos

<br>

> Añadimos la validación en la definición del formulario.

`app/cart/cart.component.ts`
```ts
  private buildForm() {
    this.form = this.fb.group({
      store: this.fb.group({
        postalCode: ['', Validators.required],
+       coupon: ['', CartValidators.validCoupon]
      }),
      selector: this.fb.group({
        product_id: '',
        quantity: 1
      }),
      cart: this.fb.array([])
    });
  }
```
<br>

> Mostramos el error en la plantilla

- Primero refactorizamos `errorRequired` por `error(name, errorCode)` para que nos sirva para otros códigos de error.

`app/cart/cart.component.ts`
```ts
  error(name: string, errorCode = 'required'): boolean {
    const control = this.parent.get(`store.${name}`);

    return control.hasError(errorCode) && control.touched;
  }
```
<br>

`app/cart/cart.component.ts`
```html
        <input
          type="text"
          placeholder="Postal code"
          formControlName="postalCode">
        <div class="error"
+         *ngIf="error('postalCode')">
          Postal code is required
        </div>

        <input
          type="text"
          placeholder="Discount coupon"
          formControlName="coupon">
+       <div class="error"
+         *ngIf="error('coupon', 'couponInvalid')">
+         Invalid coupon: 1 letter, 3 numbers
+       </div>
```
<br>


# 3 FormGroup custom validators

Vamos a definir una validación asociada al formulario, que valide que el producto seleccionado en el selector no existe ya en el carrito.

Necesitamos definir la validación en el formulario para acceder al control `cart` que contiene los elementos del carrito.

> Paso 1. Definir la nueva validación

`app/cart/validators/cart.validators.ts`
```ts
  ...

  static checkCartItemExists() {
    const exists = true;

    return exists ? { cartItemExists: true } : null;
  }
```
<br>


> Paso 2. Añadimos la validación en el formulario

`app/cart/cart.component.ts`
```ts
  private buildForm() {
    this.form = this.fb.group({
      store: this.fb.group({
        postalCode: ['', Validators.required],
        coupon: ['', CartValidators.validCoupon]
      }),
      selector: this.fb.group({
        product_id: '',
        quantity: 1
      }),
      cart: this.fb.array([])
+   }, {
+     validators: CartValidators.checkCartItemExists
+   });
  }
```
<br>

> En el componente `CartSelectorComponent` ponemos el botón disabled si se da este error.

`app/cart/components/selector/cart-selector.component.ts`
```html
        <button type="button"
          [disabled]="parent.hasError('cartItemExists')"
           (click)="onAdd()">Add to Cart</button>
```
<br>


> Implementar la validación.

`app/cart/validators/cart.validators.ts`
```ts
  static checkCartItemExists(control: AbstractControl) {
    const product = control.get('selector.product_id');
    const items = control.get('cart');

    if (!product || !items) {
      return null;
    }

    if (product.value === '') {
      return null;
    }

    const exists = items.value.some(item => +item.product_id === +product.value);

    return exists ? { cartItemExists: true } : null;
  }
```
<br>


> Ponemos el mensaje de error en el selector.

`app/cart/components/selector/cart-selector.component.ts`
```ts
export class CartSelectorComponent implements OnInit {
  ...

  get productExists() {
    return (
      this.parent.get('selector.product_id').dirty &&
      this.parent.hasError('cartItemExists')
    );
  }

}

```
<br>

`app/cart/components/selector/cart-selector.component.ts`
```html
        <button type="button"
          [disabled]="parent.hasError('cartItemExists')"
           (click)="onAdd()">Add to Cart</button>

        <div class="stock-selector__error" *ngIf="productExists">
          The product already exists in the cart
        </div>
```
<br>


# 4. Async custom validators

Hasta ahora hemos visto validaciones síncronas.

Ahora vamos a implementar una validación que necesite invocar un API y esperar la respuesta.

Vamos a añadir al mock una lista de códigos de cupones, y vamos a comprobar que el cupón introducido exista.


> Paso 1. Añadimos los datos de cupones al mock `db.json`

`mock/db.json`
``` json
{
  "cart": [
    ...
  ],
  "products": [
    ...
  ],
  "coupons": [
    { "id": "A123", "expired": false },
    { "id": "A987", "expired": true },
    { "id": "B123", "expired": false },
    { "id": "C123", "expired": false }
  ]
}
```
<br>

> Paso 2. Añadimos el servicio que obtiene los datos del API

Definimos el modelo

`app/cart/models/cart.model.ts`
```ts
export interface Coupon {
  id: string;
  expired: boolean;
}
```
<br>

Implementamos el servicio

`app/cart/services/cart.service.ts`
```ts
export class CartService {
  ...

  getCoupon(id: string) {
    return this.http.get<Coupon>(`/api/coupons/${id}`);
  }
}

```
<br>

> Paso 3. Añadimos la validación al formulario

`app/cart/cart.component.ts`
```ts
export class CartComponent implements OnInit {
    this.form = this.fb.group({
      store: this.fb.group({
        postalCode: ['', Validators.required],
        coupon: [
          '',
          CartValidators.validCoupon,
          this.validateCoupon.bind(this)
        ]
      }),
      ...
    });

  ...

  // El observable tiene que devolver el valor esperado por una validación:
  //   - null, si no hay error
  //   - { key: true } si hay error
  private validateCoupon(control: AbstractControl) {
    return this.cartService.getCoupon(control.value).pipe(
      map(coupon => {
        return coupon.expired ? {'couponExpired': true} : null;
      }),

      // Si el id no existe el api devuelve error
      // Necesitamos capturar el error para devolver un objeto
      // U N K N O W N
      catchError(
        () => of({ 'couponUnknown': true })
      )
    );
  }
}
```
<br>


> Paso 4. Mostramos el error en la plantilla

`app/cart/components/coupon/cart-coupon.component.ts`
```html
  template: `
        <div class="error"
          *ngIf="error('coupon', 'couponInvalid')">
          Invalid coupon: 1 letter, 3 numbers
        </div>
        <div class="error"
          *ngIf="error('coupon', 'couponUnknown')">
          Unknown coupon id
        </div>
        <div class="error"
          *ngIf="error('coupon', 'couponExpired')">
          Coupon expired
        </div>
  `
  ...
```
<br>

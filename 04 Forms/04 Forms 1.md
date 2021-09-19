# Reactive Forms

29. Reactive Forms setup
30. FormControls and FormGroups
31. Componetizing FormGroups
32. Binding FormControls to `<select>`
33. FormGroup collections with FormArray
34. Add items to the FormArray
35. Remove items from the FormArray
36. FormBuilder API

37. HttpService and joining Observables
38. Subscribing to valueChanges observable
39. Updating and resetting FormGroups and FormControls
40. Custom form control base
41. Implementing a ControlValueAccesor
42. Adding keyboard events to out control
43. Validators object for FormControls
44. FormControl custom validators
45. FormGroup custom validators
46. Async custom validators

# Initial code

- `/mock/db.json`
- `/package.json`
- `/proxy.conf.json`

/projects/forms

- `app/cart/components/selector/cart-selector.component.ts`
- `app/cart/components/items/cart-items.component.ts`
- `app/cart/model/cart.model.ts`
- `app/cart/services/cart.service.ts`
- `app/cart/cart.component.ts`
- `app/cart/cart.module.ts`
- `app/app.component.ts`
- `app/app.module.ts`

<br>

```bash
$ npm run start:forms
```
<br>

# xx

# 1. Reactive Forms setup

Hemos importado el módulo `ReactiveFormsModule` en el módulo `CartModule`


# 2. FormControls and FormGroups vs FormBuilder

En el fichero `app/cart/cart.component.ts` tenemos el mismo formulario construido de dos formas
- Usando `FormGroup`, `FormControl` y `FormArray`
- Usando el `FormBuilder`


# 3. Componetizing FormGroups

Vamos a dividir nuestro formulario en varios componentes:
- selector: Selección del producto y cantidad a añadir al carrito
- items: Lista de los elementos del carrito

Desde el componente padre le pasamos al componente hijo el `form`

En el componente padre `app/cart/cart.component.ts`

``` html
  <app-cart-selector [parent]="form"></app-cart-selector>
```
<br>

En el componente hijo `app/cart/components/selector/cart-selector.component.ts`

``` html
    <div class="stock-selector" [formGroup]="parent">
      <div formGroupName="selector">
        <select formControlName="product_id">
          <option value="">Select stock</option>
          <option *ngFor="let product of products"
            [value]="product.id">
            {{ product.name }}
          </option>
        </select>

        <input type="number" step="10" min="10" max="1000"
        formControlName="quantity">

        <button type="button">Add stock</button>
      </div>
    </div>
```
- Definimos un contenedor para el formulario `[formGroup]="parent"`
- Definimos un contenedor para el grupo `formGroupName="selector"`que contiene 2 campos
  - `formControlName="product_id"` (form.selector.product_id)
  - `formControlName="quantity"` (form.selector.quantity)

<br>


# 4. Binding FormControls to `<select>`

En el componente `CartSelectorComponent` podemos ver el selector de productos:

`app/cart/components/selector/cart-selector.component.ts`
``` html
    <div class="stock-selector" [formGroup]="parent">
      <div formGroupName="selector">
        <select formControlName="product_id">
          <option value="">Select stock</option>
          <option *ngFor="let product of products"
            [value]="product.id">
            {{ product.name }}
          </option>
        </select>
        ...
      </div>
    </div>
```
<br>


# 5. Use mock data: products and Cart

Vamos a pasarle los productos al selector.

## 5.1 Cargamos los datos desde el servicio `CartService`

`app/cart/cart.component.ts`
```ts
  products: Product[] = [];
  ...

  ngOnInit() {
    const cart$ = this.cartService.getCartItems();
    const products = this.cartService.getproducts();

    forkJoin([cart$, products]).subscribe( ([cart, products]) => {
      console.log('cart', cart);
      console.log('products', products);

      this.products = products;
    });
  }

```
- usamos `forkJoin` para lanzar las dos peticiones en paralelo

> `forkJoin` : Cuando todos los inner completan, se emite el último valor de cada uno
> - Si un inner da error, se pierde el valor de todos (catch)

<br>

## 5.2 Le pasamos los productos al selector

`app/cart/cart.component.ts`
```html
        <app-cart-selector
          [parent]="form"
          [products] = "products">
        </app-cart-selector>
```
<br>

`app/cart/components/selector/cart-selector.component.ts`
```ts
export class CartSelectorComponent implements OnInit {
  @Input() parent: FormGroup;
  @Input() products: Product[];
```
<br>


# 6. FormGroup collections with FormArray

## 6.1 Form Array default value

Vamos a poner un valor por defecto al `FormArray`.

`app/cart/cart.component.ts`
```ts
    this.form = this.fb.group({
      ...
      cart: this.fb.array([
        this.fb.group({
          product_id: '1',
          quantity: 2
        }),
        this.fb.group({
          product_id: '7',
          quantity: 5
        })
      ])
    });
```
<br>

## 6.2 Mostrar el formArray en el formulario

<br>

> Definimos un setter para acceder a los controles del FormArray.

`app/cart/components/items/cart-items.component.ts`
```ts
  get cartItems() {
    return (this.parent.get('cart') as FormArray).controls;
  }
```
<br>

> Definimos el `[formGroup]="parent"`

`app/cart/components/items/cart-items.component.ts`
```html
<div class="stock-product" [formGroup]="parent">
  ...
</div>
```
<br>

> Necesitamos un `formArrayName="cart"` que contenga todo
>  - Iteramos la lista de controles del `FormArray`
>  - Se crea un formGroup especial para cada fila `[formGroupName]="i"`
>  - Dentro del groupName ya tenemos cada control con `formControlName`

`app/cart/components/items/cart-items.component.ts`
```html
  <div class="stock-product" [formGroup]="parent">
+   <div formArrayName="cart">
      <div *ngFor="let item of cartItems; let i = index;">
+       <div class="stock-product__content" [formGroupName]="i">
          <div class="stock-product__name">  </div>
          <input type="number"
            step="10"
            min="10"
            max="100"
 +          formControlName="quantity">
            <button type="button">Remove</button>
        </div>
      </div>
    </div>
  </div>
```
- No vamos a editar el producto, solo tenemos un input para la cantidad

<br>

## 6.3 Añadimos el nombre del producto

<br>

> Vamos a crear un Map de productos para acceder fácilmente por id.

`app/cart/cart.component.ts`
```ts
export class CartComponent implements OnInit {
  form: FormGroup;
  products: Product[] = [];
+ productMap: ProductMap;

  ngOnInit() {
    this.loadData();
  }

  private loadData() {
    const cart$ = this.cartService.getCartItems();
    const products = this.cartService.getproducts();

    forkJoin([cart$, products]).subscribe( ([cart, products]) => {
      console.log('cart', cart);
      console.log('products', products);

      this.products = products;

 +    const map = products.map<[number, Product]>(
 +      product => [product.id, product]
 +    );
 +    this.productMap = new Map(map);
    });
  }

  ...
```
<br>

> Le pasamos el `productMap` al componente `app-cart-items`

`app/cart/cart.component.ts`
```html
        <app-cart-items
          [parent]="form"
          [productMap]="productMap">
        </app-cart-items>
```
<br>


> Añadimos un Input en `CartItemsComponent`

`app/cart/components/items/cart-items.component.ts`
```ts
export class CartItemsComponent implements OnInit {
  @Input() parent: FormGroup;
+ @Input() productMap: ProductMap;
  ...
}
```
<br>

> Y definimos un método para acceder al nombre del producto

```ts
  <div class="stock-product__name">{{ getProductName(item) }}</div>

  getProductName(control: FormControl) {
    if (this.productMap) {
      const id = +control.value.product_id;
      return this.productMap.get(id).name;
    }

    return '';
  }
```
<br>

## 6.4 Add items to the FormArray

<br>

> Añadimos dos métodos
> -  `createItem` para crear cada elemento que añadiremos al FormArray
> - `addItem` para añadir ese elemento al FormArray

`app/cart/cart.component.ts`
```ts
  private createItem(item: Item) {
    return this.fb.group({
      product_id: item.product_id,
      quantity: item.quantity || 10
    });
  }

  private addItem(item: FormGroup) {
    const control = this.form.get('cart') as FormArray;
    control.push(item);
  }
```
<br>

> Añadimos al FormArray los valores iniciales obtenidos del api

`app/cart/cart.component.ts`
```ts
  private buildForm() {
    this.form = this.fb.group({
      selector: this.fb.group({
        product_id: '',
        quantity: 10
      }),
+     cart: this.fb.array([])
    });
```
<br>

`app/cart/cart.component.ts`
```ts
  private loadData() {
    const cart$ = this.cartService.getCartItems();
    const products = this.cartService.getproducts();

    forkJoin([cart$, products]).subscribe( ([cart, products]) => {

      this.products = products;

      const map = products.map<[number, Product]>(
        product => [product.id, product]
      );
      this.productMap = new Map(map);

+     cart.forEach(item => {
+       this.addItem(this.createItem(item));
+     });
    });
  }
```
<br>


## 6.5 Add items selected with `app-cart-selector`

<br>

> Añadimos un @Output() para emitir los valores seleccionados

`app/cart/components/selector/cart-selector.component.ts`
```html
        <button type="button" (click)="onAdd()">Add to Cart</button>
```
<br>

`app/cart/components/selector/cart-selector.component.ts`
```ts
export class CartSelectorComponent implements OnInit {
  @Output() added = new EventEmitter();

  constructor() { }

  ngOnInit() { }

  onAdd() {
    this.added.emit(this.parent.get('selector').value);
  }
}

```
<br>

> Capturamos este evento y lo añadimos al FormArray

`app/cart/cart.component.ts`
```html
        <app-cart-selector
          [parent]="form"
          [products] = "products"
          (added)="onAddItem($event)">
        </app-cart-selector>
```
<br>

`app/cart/cart.component.ts`
```ts
  onAddItem(item: Item) {
    this.addItem(this.createItem(item));
  }
```
<br>


## 6.6 Remove items from the FormArray

> Añadimos un Output en `app-cart-items` para emitir el elemento del formAray a eliminar.
> - Necesitamos el indice del array

`app/cart/components/items/cart-items.component.ts`
```ts
  template: `
    <button type="button" (click)="onRemove(i)">Remove</button>
  `

export class CartItemsComponent implements OnInit {
  @Input() parent: FormGroup;
  @Input() productMap: ProductMap;
  @Output() removed = new EventEmitter();

  ...

  onRemove(index) {
    this.removed.emit(index);
  }
}
```
<br>

> En el componente `CartComponent` recogemos el valor y eliminamos la fila del formArray
> - Utilizamos el método `FormArray.removeAt(index)`

`app/cart/cart.component.ts`
``` ts
  template: `
        ...
        <app-cart-items
          [parent]="form"
          [productMap]="productMap"
          (removed)="removeItem($event)">
        </app-cart-items>
        ...
  `
  ...

  removeItem(index) {
    const control = this.form.get('cart') as FormArray;
    control.removeAt(index);
  }

```
<br>

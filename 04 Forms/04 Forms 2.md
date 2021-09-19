# Reactive Forms

38. Subscribing to valueChanges observable
39. Updating and resetting FormGroups and FormControls
40. Custom form control base
41. Implementing a ControlValueAccesor
42. Adding keyboard events to out control
43. Validators object for FormControls
44. FormControl custom validators
45. FormGroup custom validators
46. Async custom validators

# 1. Subscribing to valueChanges observable

Vamos a mostrar el total del carrito.
- Recalculamos el total, suscribiendonos al valueChanges del formulario.

> Definimos una propiedad para guardar y mostrar el total.

`app/cart/cart.component.ts`
``` ts
        <app-cart-items> ...
        </app-cart-items>

      <div class="stock-inventory__price">
        Total: {{ total | currency:'EUR':true }}
      </div>

export class CartComponent implements OnInit {
  ...
  total: number;

}
```
<br>

> Añadimos un método para calcular el total del carrito.

`app/cart/cart.component.ts`
``` ts
export class CartComponent implements OnInit {
  ...
  total: number;

  ...

  private calculateTotal(items: Item[]) {
    console.log(items, this.productMap);
    const total = items.reduce((prev, next) => {
      const totalNext = next.quantity * this.productMap.get(+next.product_id).price;
      return prev + totalNext;
    }, 0);

    this.total = total;
  }
}
```
<br>

> Calculamos el total al inicio,
> - y cada vez que se modifica el valor del `FormArray`

`app/cart/cart.component.ts`
```ts
  private loadData() {
    const cart$ = this.cartService.getCartItems();
    const products = this.cartService.getproducts();

    forkJoin([cart$, products]).subscribe( ([cart, products]) => {
      ...

      this.calculateTotal(this.form.get('cart').value);

      this.form.get('cart').valueChanges.subscribe(
        value => this.calculateTotal(value)
      );
    });
  }

```
<br>

# 2. Updating and resetting FormGroups and FormControls

Vamos a resetear el control cada vez que añadimos un producto al carrito.

`app/cart/cart-selector.component.ts`
``` ts
 onAdd() {
    this.added.emit(this.parent.get('selector').value);
    this.parent.get('selector').reset({
      product_id: '',
      quantity: 1
    });
  }
```
<br>

El reset, restablece los estados del control a su valor inicial: ng-valid, untouched, pristine.

Si en lugar de usar reset, usamos patchValue() o setValue() se modifican los valores, pero se mantiene el estado actual.

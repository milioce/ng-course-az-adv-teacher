# Indice

10.6 Configurable NgModules
10.7 Zones and NgZone


# 6.0 Initial files

Tenemos el código de la aplicación de una tienda en la carpeta `app/store`

- `store.config.ts` un token con los datos de una tienda que hay que pasar en los headers
- `store.module.ts` un módulo con un provider que da valor a esos valores de configuración
- `store.service.ts` un servicio que coge los valores de configuración y los añade a los headers
- `store.component.ts` llama al método del servicio y muestra los datos

<br>

> Descomentar el import del módulo `StoreModule` en el `AppModule`

`app/app.module.ts`
``` ts
  imports: [
    ...
    // StoreModule
  ],
```
<br>

> Para ver la demo ponemos `app-store` en el template del `AppComponent`.

`app/app.component.ts`
``` html
    <app-store></app-store>
```
<br>

Vamos en que se están añadiendo los headers al ejecutar las peticiones.

<br>

> Objetivo. Módulo configurable

Nuestro objetivo es hacer el módulo configurable, para que al importarlo le podamos pasar la configuración:

`app/app.module.ts`
``` ts
    StoreModule,
    /*
    StoreModule.forRoot({
      storeId: 10000,
      storeToken: 'AF02-0124-5565-1234'
    })
    */
```
<br>

# 6.1 Crear móduilo `StoreModule` configurable

Creamos un método `static forRoot(config): ModuleWithProviders<T>`
- Recibe como parametro la configuración
- Devuelve una definición de módulo con providers
- Define un provider con la configuración recibida para poder acceder a ella

`app/store/store.module.ts`
``` ts
export class StoreModule {
  static forRoot(config: StoreConfig): ModuleWithProviders<StoreModule> {
    return {
      ngModule: StoreModule,
      providers: [
        { provide: STORE_CONFIG, useValue: config }
      ]
    }
  }
}
```
- Configuración opcional => `useValue: config || {}`

<br>

Quitamos el provider que teníamos definido dentro de `@Component`

`app/store/store.module.ts`
``` ts
@NgModule({
  declarations: [ StoreComponent ],
  imports: [ CommonModule ],
  exports: [ StoreComponent ],
  providers: [
    // {
    //   provide: STORE_CONFIG,
    //   useValue: { storeId: 10000, storeToken: 'AF02-0124-5565-1234' }
    // }
  ],
})

```
<br>

# 6.2 Importar el módulo con la configuración.

En el `AppModule` vamos a importar el módulo `StoreModule` usando el método `forRoot` para pasarle la configuración.

`app/app.module.ts`
``` ts
@NgModule({
  declarations: [AppComponent],
  imports: [
    ...,
    StoreModule.forRoot({
      storeId: 10000,
      storeToken: 'AF02-0124-5565-1234'
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
```
<br>

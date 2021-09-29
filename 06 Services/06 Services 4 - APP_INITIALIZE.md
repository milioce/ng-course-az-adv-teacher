# 6.1 APP_INITIALIZE

ES un `DI Token` que podemos usar para definir una función de inicialización,
que se inyecta durante la inicialización de la aplicación.

Si devuelve un observable o una promesa, la inicialización no termina hasta que se complete el observable
o se resuelva la promesa.

Sepuede usar para cargar ficheros de traducción, configuración externa, etc.

<br>

# 6.2 Ejemplo 1. Factory

`app/core/core.module.ts`
```ts
const appInitializer1 = () => {
  console.log('appInitializer1');
  return Promise.resolve(true);
}

@NgModule({
  ...
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: () => appInitializer1,
      multi: true
    },
  ]
})
```
<br>

# 6.3 Ejemplo 2. Factory con dependencias

`app/core/core.module.ts`
```ts
const appInitializer = (http: HttpClient) => {
  console.log('appInitializer2', http);
  return () => {
    return http.get('/api/countries')
      .toPromise()
      .then(data => {
        console.log('data', data);
        return true;
      });
  }
}

@NgModule({
  ...
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: appInitializer,
      deps: [HttpClient],
      multi: true
    }
  ]
})
```
<br>

# 6.3 Ejemplo 3. Inicializar configuracion externa

Vamos a crear un fichero en la carpeta `assets` con la configuración de la aplicación. Este fichero se puede sobrescribir durante una pipeline de despliegue.

<br>

## 6.3.1 Crear fichero de configuración
---

Creamos el fichero en `src/assets/config.json`


`src/assets/config.json`
```json
{
  "apiHost": "/api",
  "level": 1,
  "apiKey": "1234-5678-1234-5678",
  "default": "es"
}
```
<br>

## 6.3.2 Crear modelo
---

Creamos un fichero `app/core/core.model.ts`
- Creamos la clase `AppConfig`
- Creamos un token para inyectar un provider

`app/core/core.model.ts`
```ts
import { InjectionToken } from "@angular/core";

export class AppConfig {
  apiHost = '';
  level = 0;
  apiKey = '';
  default = 'es';
}

export const APP_CONFIG = new InjectionToken<AppConfig>('APP_CONFIG');
```
<br>

## 6.3.3 Definimos el provider `APP_CONFIG`
---

Definimos un `provider` que creará una instancia de la clase `AppConfig`.

En nuestra aplicación inyectaremos este provider para acceder a la configuración.

`app/core/core.module.ts`
```ts
  providers: [
    {
      provide: APP_CONFIG,
      useClass: AppConfig
    },
```
<br>

## 6.3.4 Inicializar el `AppConfig`

- Le pasamos el servicio Injector para acceder a las dependencias.

`app/core/core.module.ts`
```ts
const appInitializer = (_injector: Injector) => {
  console.log('appInitializer2');
  return () => {
    const httpClient = _injector.get(HttpClient);
    const appConfig = _injector.get(APP_CONFIG);

    return httpClient.get('/assets/config.json')
      .toPromise()
      .then((config: AppConfig) => {
        console.log('config.json', config);
        appConfig.apiHost = config.apiHost;
        appConfig.apiKey = config.apiKey;
        appConfig.default = config.default;
        appConfig.level = config.level;

        return true;
      }).
      catch(() => {
        console.log('Error al recuperar el fichero de configuración');
        return true;
      });
  }
}
```
<br>

En el provider ponemos el `Injector` como dependencia

`app/core/core.module.ts`
```ts
  ...
    {
      provide: APP_INITIALIZER,
      useFactory: appInitializer,
      deps: [Injector],
      multi: true
    }
```
<br>
# 4. Forms

# 1 Instalar Mock Server

## 1.1 Instalar `json-server`
---

Se puede instalar json-server globalmente y usarlo en todos los proyectos
o se puede instalar localmente en el proyecto que lo necesitemos.

```bash
$ npm install json-server --save-dev
```
<br>

## 1.2 Creamos los ficheros de configuración

Definimos el fichero de datos

`mock/db.json`
``` json
{
  "cart": [
    { "product_id": 1, "quantity": 1 },
    { "product_id": 3, "quantity": 50 }
  ],
  "products": [
    { "id": 1, "price": 898, "name": "Portátil Intel Core i7" },
    { "id": 2, "price": 120, "name": "Monitor 23.6\" LED FullHD" },
    { "id": 3, "price": 39, "name": "Teclado Gaming" },
    { "id": 4, "price": 4.90, "name": "Ratón Gaming" },
    { "id": 5, "price": 70, "name": "Impresora Wifi" },
    { "id": 6, "price": 21.90, "name": "Tinta impresora compatible" },
    { "id": 7, "price": 2.25, "name": "Cable USB 2.0" },
    { "id": 8, "price": 28, "name": "Pulsera de actividad" }
  ]
}
```
<br>

## 1.3 Definir un script para iniciar el server

Añadimos un comando en la sección `scripts` del fichero `package.json`

`package.json`
```json
  "scripts": {
    "mock:server": "npx json-server --watch mock/db.json",
  },
```
<br>

Y ejecutamos el comando

```bash
npm run mock:server
```
- Arranca un server en localhost:3000, con dos recursos
  - `http://localhost:3000/cart`
  - `http://localhost:3000/products`

<br>

# 1.4 Configurar el proxy en angular

Vamos a usar un proxy en angular para poder usar `http://localhost:4200/api` y que nos redirija a `http:/localhost:3000`

- creamos el fichero `proxy.conf.json` en el raiz

`proxy.conf.json`
```json
{
  "/api": {
    "target": "http://localhost:3000",
    "secure": false,
    "changeOrigin": false,
    "pathRewrite": {
      "^/api": ""
    }
  }
}
```
<br>

Añadimos un comando al `package.json`
```json
"serve-forms": "ng serve --project forms --proxy-config proxy.conf.json",
```

Probamos el comando ejecutando
```bash
npm run serve-forms
```

## 1.5 Ejecutamos los dos comandos concurrentemente.

Instalamos el paquete `concurrently`
```bash
npm install concurrently --save-dev
```

Añadimos un comando al `package.json`
```json
    "start:forms": "concurrently \"npm run mock:server\" \"npm run serve-forms\"",
    "serve-forms": "ng serve --project forms --proxy-config proxy.conf.json",
```
<br>

# 1.6 Arrancar el proyecto `forms`

```bash
npm run start:forms
```
<br>

# 2. Initial files
---
---
---

<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

## xxx
`app/cart/xxx.ts`
```ts
```
<br>

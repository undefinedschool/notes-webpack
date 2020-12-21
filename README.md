# Notas sobre Webpack

### 👉 Ver [todas las notas](https://github.com/undefinedschool/notes)

## Intro

Webpack es un _module bundler_, su tarea es analizar el código de nuestra aplicación, detectar los diferentes módulos que el mismo importa/exporta, aplicar (opcionalmente) las transformaciones necesarias y empaquetar todo el código en un único archivo `.js` y referenciarlo en el `index.html`, para que este pueda ser ejecutado en los browsers luego.

## Bundling

Uno de los problemas que resuelve Webpack entonces, es la necesidad que teníamos previamente de linkear todos los scripts de librerías y módulos que utilizáramos en el `index.html`, en el orden correcto. 

Los problemas que generaba esto eran: 

- dependencia de servidores de CDN (librerías)
- problemas para el versionado: si queremos actualizar alguna librería, tenemos que modificar manualmente los números de versiones en el `src`, lo que es poco práctico y propenso a errores
- cargar los scrips en el orden incorrecto (si tienen dependencias entre si)

```html
<!-- Linkeando scripts como antes -->
<body>
  ...
  <script src="libs/react.min.js"></script>
  <script src="src/admin.js"></script>
  <script src="src/api.js.js"></script>
  <script src="src/auth.js"></script>
</body>
```

```html
<!-- Con Webpack -->
<body>
  ...
  <script src="dist/bundle.js"></script>
</body>
```

## Transformaciones y optimizaciones

Aparte de resolver el problema del _bundling_, Webpack permite, a través de diferentes plugins y _loaders_, aplicar diferentes transformaciones (ej: transpilar TS, o código ES6+, agregar Babel, etc) y optimizaciones a nuestro código (ej: code splitting, image opt, etc).

### Loaders

Por defecto, Webpack sólo puede analizar y procesar los archivos `.js` y `.json` de nuestro código. Probablemente necesitemos incluir estilos (CSS) e imágenes. Es por este motivo que existen los _loaders_, que le permiten a Webpack procesar más tipos de archivos, para realizar diferentes transformaciones, optimizaciones e incluirlos en el bundle final.

Para cada loader, tenemos que especificar sobre qué archivos se va a aplicar (podemos usar regex).

Los loaders se instalan a través de `npm`, por ejemplo, para procesar [SVGs](https://v4.webpack.js.org/loaders/svg-inline-loader/) y [CSS](https://webpack.js.org/loaders/css-loader/)

```
npm i -D svg-inline-loader css-loader
```

Si además queremos inyectar los estilos directamente en el DOM, podemos usar [`style-loader`](https://webpack.js.org/loaders/style-loader/)

```
npm i -D style-loader
```

### Plugins

A diferencia de los _loaders_, que se utilizan para procesar y transformar archivos mientras generamos el bundle, **los _plugins_ le permiten a Webpack ejecutar ciertas tareas después de que el bundle fue creado**. Por ejemplo, [`HtmlWebpackPlugin`](https://webpack.js.org/plugins/html-webpack-plugin/): genera un `index.html`, lo coloca en `/dist` (o en cualquier otro destino que definamos en `output`), agregándole un `<script>` tag que referencia al bundle creado. Otro plugin útil es [`EnvironmentPlugin`](https://webpack.js.org/plugins/environment-plugin/), que nos permite configurar el entorno de producción para el deployment.

Para agregar plugins, debemos instalarlos (al igual que los loaders) y setearlos en la propiedad `plugins` de la config

```
npm i -D html-webpack-plugin
```

```js
// webpack.config.js
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  module: {
    rules: [ ... ]
  },
  output: { ... },
  plugins: [
    new HtmlWebpackPlugin(),
    new webpack.EnvironmentPlugin({
      'NODE_ENV': 'production'
    })
  ]
}
```

## HMR

El Hot Module Replacement/Reloading permite agilizar el proceso de desarrollo, al dejarnos ver los cambios de forma instantánea en el browser, sin necesidad de refrescar el mismo. El mismo sólo estará disponible en el entorno de desarrollo, no así en producción.

## Setup

Instalar Webpack y su CLI como _dependencias de desarrollo_ en el proyecto:

```
npm i -D webpack webpack-cli
```

### Config 

Crear el archivo `webpack.config.js` para guardar las opciones de configuración.

```js
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.svg$/,          // aplicar esta transformación a todos los archivos .svg
        use: 'svg-inline-loader' // loader que vamos a usar
      },
      {
        test: /\.css$/,                       // aplicar esta transformación a todos los archivos .css
        use: [ 'style-loader', 'css-loader' ] //  en este caso usamos 2 loaders, por eso el array. El orden importa, webpack los procesa en orden inverso
      }
    ]
  }
}
```

En este archivo de config deberemos indicar cosas como:

- **entry point** de nuestra app (definido en `entry`)
- **transformaciones** que querramos aplicar (definidas en las `rules` dentro de `module`)
- **destino**, con la ubicación del archivo empaquetado (definido en `output`)

Agregando el `babel-loader` para transpilar JS ES6+

```
npm i -D babel-loader
```

```js
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.svg$/,          
        use: 'svg-inline-loader' 
      },
      {
        test: /\.css$/,                       
        use: [ 'style-loader', 'css-loader' ]
      {
        test: /\.(js)$/,         // aplicar esta transformación a todos los archivos .js
        use: 'babel-loader'
      }
    ]
  }
}
```

Finalmente, para agregar el destino, el `output`, que es un objeto con las siguientes propiedades

```js
output: {
  path: ...     // ubicación de /dist, el directorio donde se va a ubicar el bundle
  filename: ... // nombre del bundle
}
```

```js
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.svg$/,          
        use: 'svg-inline-loader' 
      },
      {
        test: /\.css$/,                       
        use: [ 'style-loader', 'css-loader' ]
      {
        test: /\.(js)$/,         // aplicar esta transformación a todos los archivos .js
        use: 'babel-loader'
      }
    ]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'index.bundle.js`
  }
}
```

### `tl;dr`

1) Webpack analiza el archivo definido como entry point
2) Detecta y examina todos los imports/exports del mismo (o `requires`, si estamos usando CommonJS) y genera un [_dependency graph_](https://webpack.js.org/concepts/dependency-graph/), para saber qué módulos dependen de cuáles otros y en qué orden debe insertarlos en el bundle final
3) Comienza a crear el bundle y a medida que va agregando los scripts, aplica las transformaciones definidas en la config y agrega el output correspondiente
4) Finalmente ubica el bundle final según `output` definido en la config

## Dev & Production mode

Podemos realizar el proceso de preparar nuestro bundle para producción (setear el `NODE_ENV`, minificar el código, etc) manualmente, en el caso de que querramos optimizar algo específico. Pero para simplificar y ahorrar tiempo, podemos usar el `mode` de la config y setearlo en _production_, que va a setear ciertos parámetros por default (como los mencionados antes).

```js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  module: {
    rules: [ ... ]
  },
  output: { ... },
  plugins: [
    new HtmlWebpackPlugin()
  ],
  mode: process.env.NODE_ENV === 'production' ? 'production' : 'development'
}
```

El mode _development_ de webpack deshabilita las optimizaciones mencionadas y activa el HMR para desarrollar localmente.

## DevServer

El [DevServer](https://webpack.js.org/configuration/dev-server/) nos permite levantar un servidor local de desarrollo y carga los archivos en memoria, evitándonos esperar a que se complete el build cada vez que guardamos cambios, para poder ver los mismos en el browser. Este módulo además es el que incluye el HMR (live reloading). Para usarlo, lo instalamos con

```
npm i -D webpack-dev-server
```

## Ejecutar Webpack desde los scripts

En nuestro `package.json`, agregamos los scripts de _build_ (bundle optimizado para producción) y _dev_ (este último levanta un server local con [HMR](https://webpack.js.org/concepts/hot-module-replacement/))

```json
scripts: {
  build: "NODE_ENV='production' webpack",
  dev: "webpack-dev-server"
}
```

y luego lo corremos con `npm run build`.

## Más info

- [Documentación oficial](https://webpack.js.org/concepts/)
- [webpack: The Core Concepts](https://webpack.academy/p/the-core-concepts)
- [Web Fundamentals](https://webpack.academy/p/web-fundamentals)

# Laravel Auth - Backoffice & Frontoffice

## Dopo aver creato il progetto

- Installiamo il pacchetto laravel ui ```composer require laravel/ui:^2.4```
- Creiamo lo scaffolding Bootstrap con autenticazione ```php artisan ui bootstrap --auth```
- Lanciare ```npm install && npm run dev```

## Eseguiamo le migration

Verifichaimo di aver creato e collegato correttamente il database, quindi lanciamo ```php artisan migrate```

## Separazione Backoffice e Frontoffice
Laravel crea già un **HomeController**, possiamo eliminarlo.
Creiamo quindi un **HomeController** sotto il namespace Admin che gestirà la pagina di atterraggio dopo il login 
```php artisan make:controller Admin/HomeController ```
 
### Definizione rotte 🗺
definiamo ora le rotte per la parte di backoffice, raggruppandole con namespace, prefisso, middleware auth e name.
```php
// routes/web.php

// Rotte Autenticazione
Auth::routes();

// Rotte area Admin
Route::middleware('auth')->namespace('Admin')->prefix('admin')->group(function() {
    Route::get('/home', 'HomeController@index')->name('home');
});

// Rotte pubbliche
Route::get('/', function () {
    return view('front');
});
```

### Modifica path della costante HOME ↩️
Cambiamo l'url di redirect dopo il login (al posto di `/home`, scriviamo `/admin`)
```php
// app/Providers/RouteServiceProvider.php

class RouteServiceProvider extends ServiceProvider
{
    // ...

		/**
     * The path to the "home" route for your application.
     *
     * @var string
     */
    public const HOME = '/admin/home';

		// ...
}
```
### Organizzazione views 🗂
Laravel ci crea già una struttura di partenza che comprende il layout **app.blade.php** che utilizzaremo come base per le nostre pagine del backoffice e una cartella **auth** con all'interno tutte le views per l'autenticazione.

Possiamo quindi creare una sottocartella `resources/views/admin/` per gestire tutte le views delle nostre CRUD di  **backoffice** e un file **front.blade.php** dove verrà rendirizzata la nostra applicazione in Vuejs.

### Organizzazione css/js 💅
Siccome avremo bisogno di due file css e js distinti fra frontoffice e backoffice inziamo rinominando gli attuali file **app.scss** e **app.js** in **admin.scss** e **admin.js**.
Ora aggiungiamo lo scaffolding di vue:
```php artisan ui vue```
Verranno nuovamente creati i file **app.scss** e **app.js** che potremmo rinominare in **front.scss** e **front.js**.
Aggiungiamo il nostro componente principale **App.vue** sempre nella cartella `resources/js`

N.b Il file **front.js** può essere strutturato in questo modo:
```js
window.axios = require('axios');
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
window.Vue = require('vue');

import App from './App.vue';

const app = new Vue({
    el: '#app',
    render: h => h(App)
});
```

#### Compilazione degli assets
Essendo che abbiamo rinominato e aggiunto file css e js dovremmo aggiornare il file webpack.mix.js per la corretta compilazione nella cartella public.
```js
// webpack.mix.js

// compilazione assets per backoffice
mix.js('resources/js/admin.js', 'public/js')
    .sass('resources/sass/admin.scss', 'public/css');

// compilazione assets per frontoffice
mix.js('resources/js/front.js', 'public/js')
    .sass('resources/sass/front.scss', 'public/css');
```

Ora lanciamo ```npm install && npm run dev``` per completare l'installazione e ricompilare gli assets.

## Crud time 🥕

Arrivati a questo punto possiamo dedicarci alla creazione delle nostre **CRUD** per ogni singola entità.
Ricordandoci di aggiugnere il namespace Admin in fase di creazione del nostro controller.
Es.
```php artisan make:controller Admin/PostController --resource ```

## Api 🐝
Creiamo il controller, sotto al namespace *Api*, per le entità per le quali vogliamo esporre le api.
Es.
```php artisan make:controller Api/PostController```

Nel file **api.php** aggiungiamo le nostre rotte api.
```php
// routes/api.php

// api/posts
Route::get("/posts", "Api\PostController@index");
```

Ricordiamoci che i metodi del controller dovranno restiuire un Json e non una view, pertanto utiliziamo il metodo `response()->json();`
Es.
```php
// App/Http/Controllers/Api/PostController.php

public function index()
{
    // tutti i posts
    $posts = Post::all();

    return response()->json($posts);
}
```

## Vue Router 
Installiamo il pacchetto vue router ```npm i vue-router@3.5.3```

### Gestire le rotte
Rindirizzo tutto le richeste verso la pagina vue
```php
// /routes/web.php

Route::get("{any?}", function() {
    return view("front");
})->where("any", ".*");
```

Aggiunto il tag nel main
```<router-view></router-view>```

Creo il file delle rotte per il Vue router
```js
// resources/js/router.js

import Vue from "vue";
import VueRouter from "vue-router";

Vue.use(VueRouter);

import Home from "./pages/Home";
const router = new VueRouter({
    mode: "history",
    routes: [
        {
            path: "/",
            name: "home",
            component: Home
        },
    ]
});

export default router
```
### Importiamo il file router.js e aggiungiamolo all'istanza Vue
```js
// resources/js/front.js

window.axios = require('axios');
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
window.Vue = require('vue');

import App from './App.vue';

// aggiungiamo l'import del file router.js
import router from "./router";
const app = new Vue({
    el: '#root',
    render: h => h(App),
		// aggiungiamo l'oggetto router all'istanza Vue
		router
});
```

### Link fra le pagine
```<router-link :to="{ name: routeName }">Label</router-link>```
Con parametro dinamico
```<router-link :to="{ name: routeName, params: { slug:slug } }">Label</router-link>```


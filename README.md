
Primeros pasos Laravel 5
------------------------

**Creacion del proyecto**

Laravel 5 crea el proyecto con el comando:

`composer create-project laravel/laravel --prefer-dist 5.* nombreproyecto
`

Entre sus diferencias mas notorias esta el hecho de que (al igual que symfony2) crea un archivo env donde estan los datos de tu base de datos y tu gestor de emails.



**Pruebas en servidor propio**

Para hacer pruebas sin tener que configurar un servidor en el momento, utilizamos el comando:

`php artisan serve --host=192.168.64.166
`

O la ip que tu quieras ponerle.



**Migraciones**

para comenzar a migrar las tablas, utilizamos

`php artisan make:migration create_descriptions_table --create=descriptions
`

Esto automaticamente generara archivos migrate que lucen como el siguiente:

```js
public function up()
{
    Schema::create('descriptions', function(Blueprint $table){
      $table->increments('id');
      $table->timestamps();
      //agregado personalmente
      $table->integer('product_id')->unsigned();
      $table->integer('product_id')->references('id')->on('products');
      $table->text('body');
    });
}

public function down()
{
    Schema::drop('descriptions');
}
```

Una vez esten terminadas las migraciones, ejecutamos:

`php artisan migrate
`

En caso de que por algun motivo quieras regresar en tu ultima migracion, deshacerla por decirlo de otro modo, puedes utilizar:

`php artisan migrate:rollback
`

Si lo que quieres es regresarlas todas (en caso de haber hecho varias ejecuciones de migrate), puedes utilizar:

`php artisan migrate:reset
`

Y tambien esta la opcion de refrescar que es ponerse al dia con las migraciones que esten pendientes por ejecutarse:

`php artisan migrate:refresh
`

Cuando queramos crear un semillero para la BD, utilizamos la instrucción:

`php artisan make:seeder DescriptionTableSeeder
`

Hay que agregar las lineas a los llamados en la clase DatabaseSeeder asi:

```js
Model::unguard();
//Agregado
$this->call(DescriptionTableSeeder::class);
//Fin Agregado
Model::reguard();
```

y en cada TableSeeder donde quieras agregar datos a la base de datos, escribes:

```js
public function run()
{
  $descriptions = ['desc1','desc1','desc1',];
  array_map(function ($body){
      $now = date('Y-m-d H:i:s', strtotime('now'));
      DB::table('products')->insert([
          'body' => $body,
          'created_at' => $now,
          'updated_at' => $now,
      ]);
  }, $descriptions);

}
```

Para ejecutar las clases seed, y crear esos inserts en la BD escribimos:

`php artisan db:seed
`

Si queremos ejecutar una base o tabla especifica, utilizamos el comando

`php artisan db:seed --class=DescriptionTableSeeder
`


**Modelos**

Para cuando queramos crear nuestros modelos en el proyecto Laravel, vamos a utilizar la instrucción:

`php artisan make:model nombremodelo
`

Eso nos creara el modelo en la carpeta Providers de app.

Si tenemos por ejemplo dos modelos y uno tiene un foraneo de otro (Product <- Description)

agregaremos la funcion en Product.PHP:

```js
namespace App;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
  public function descriptions()
  {
      return $this->hasMany(Description::class);
  }
}
```

Esto lo hacemos para que Eloquent conozca que un valor de la base de datos es un foraneo

Para hacer la contrapartida en Description.php

```js
namespace App;

use Illuminate\Database\Eloquent\Model;

class Description extends Model
{
  public function product()
  {
      return $this->belongsTo(Product::class);
  }
}
```



**Rutas**

La parte de rutas queda alojada en la direccion app/Http/routes.php


Para listar la cantidad de rutas:

`php artisan route:list
`

las rutas pueden ser algo así

```js
Route::get('/', function(){
    return view('welcome');
});

Route::get('descriptions', function(){
    return App\Description::all();
});
```

la ruta descriptions retornara todas las descripciones que esten alojadas en la BD.

Cuando queramos usar esa ruta en otras partes de la aplicacion, como por ejemplo en las vistas, es necesario agregar la siguiente modificación:

```js
Route::get('descriptions',['as'=>'products', function(){
    return App\Description::all();
}]);
```

En las rutas existe una instruccion llamada group que me permite contener varias rutas dentro de una misma, ya sea para darle algun prefijo a la direccion web o para hacer verificaciones de seguridad. Para agregarle el prefijo api, haremos el siguiente cambio


```js
Route::group(['prefix' => 'api'], function(){
  Route::get('descriptions',['as'=>'products', function(){
    return App\Description::all();
  }]);
});
```

Sin embargo cuando requieras poner las rutas de un controlador, deberas tener en cuenta la instruccion resource:

```js
Route::resource('products', 'ProductController');
```

Pero si queremos limitar las opciones a escoger en el controlador, haremos la siguiente modificación:

```js
Route::resource('products', 'ProductController', ['only'=>['index, store']]);
```



**Testeo**

En nuestro directorio app/tests se alojan las pruebas tests de nuestra aplicacion, vamos a crear una prueba en el archivo ExampleTest.php:


```js
public function testDescriptionsList()
{
    $this->get(route('products'))
        ->assertResponseOk();
}
```

Esto nos confirmara que la ruta que hemos creado, tenga una respuesta satisfactoria (un codigo 200).

Para poder ejecutarla, escribiremos en consola, el comando:

`phpunit
`



**Controladores**

Para crear controladores de una clase, utilizaremos la siguiente instrucción (preferiblemente con la primera letra del modelo en mayuscula):

`php artisan make:controller NombremodeloController
`

Esto creara nuestros controladores en el directorio app/Http/Controllers


Hay que añadir tambien la siguiente linea use para que se puedan traer los objetos dell modelo Description o el que queramos:

```js
use App\Http\Descriptions

public function index()
{
    return Description::all();
}
```

Cuando queramos crear consultas en un valor Foraneo, escribimos la siguiente funcion en la clase de la cual es el foraneo:

```js
public function scopeOfProduct($query, $productId)
{
    return $query->where('product_id', $productId);
}
```

Mientras en el controlador utilizamos la linea:

```js
public function index()
{
    return Description::ofProduct($productId)->get();
}
```

Pero mostrar toda esa cantidad de datos podria traernos problemas cuando tenemos tablas con miles de datos, por lo cual siempre es bueno paginar para no traer cantidades grandes de datos y dividir mejor la información:

```js
public function index()
{
    return Description::ofProduct($productId)->get()->paginate();
}

//en el otro controlador
public function index()
{
    return Description::paginate();
}
```

Esto agregara otros metadatos cuando hagas la consulta en tu navegador.



**Factory - Fabrica**

Laravel viene con un directorio database/Factories donde se alojan las fabricas que se han creado para el proyecto.

Estas Fabricas sirven para hacer inserts en una BD que se crea en memoria para testear que realmente si se esta añadiendo datos correctamente.

En el archivo modelFactory.php puedes crear las siguientes fabricas:

```js
$factory->define(App\Description::class, function($faker){
    return [
        'name' => $faker->word,
        //tambien puede ser
        'body' => $faker->text,
    ];
});
```

Donde tienes que agregar los campos de tu modelo.
De aqui, eso nos llevara a una nueva etapa en los testeos.



**Testeo (Parte 2)**

En nuestra clase de ExampleTest, modificaremos la funcion para recibir datos de una fabrica y revisarlos.

Si aun no hemos agregado la instruccion use DatabaseTransactions; tenemos que agregarla

```js
use DatabaseTransactions;

public function testProductsList()
{
    $products = factory(\App\Product::class, 3)->create();

    $this->get(route('api.products.index'))
        ->assertResponseOk();

    array_map(function ($product){
        $this->seeJson($product->jsonSerialize());
    }, $products->all());
}
```

De esta manera es posible testear inserts y luego verificar si realmente esta guardando los datos.



**Guardar datos en la BD**

Para guardar datos en el controlador, utilizamos la funcion store, en esta añadimos el request como parametro (si no lo tiene aun)
y luego escribimos lo siguiente:

```js
public function store(Request $request)
{
    return Product::create([
        'name' => $request->input('name')
    ]);
}
```

Sin embargo, esto no nos guardara ningun valor, para verificarlo podemos hacer un *test* que nos verifique así:

```js
public function testProductCreaction()
{
    $product = factory(\App\Product::class)->make(['name'=>'beets']);

    $this->post(route('api.products.store'), $product->jsonSerialize(), $this->jsonHeaders())
        ->seeInDatabase('products', ['name'=>$product->name])
        ->assertResponseOk();

}
```

Pero nos devolvera error, esto porque en el modelo de la BD aun no hemos establecido que cambos son llenables.


Asi que en el modelo, escribiremos la linea.

```js
    public $fillable = ['name'];
```



**Actualizar datos en la BD**

En este caso, crearemos una prueba test para el caso en el que se quiera modificar, recordando que cuando actualizas un valor, se utiliza la funcion PUT:

```js
public function testProductUpdate()
{
    $product = factory(\App\Product::class)->make(['name'=>'beets']);
    $product->name = 'feets';

    $this->put(route('api.products.update', ['products' => $product->id]), $product->jsonSerialize(), $this->jsonHeaders())
        ->seeInDatabase('products', ['name'=>$product->name])
        ->assertResponseOk();
}
```

Si lo probamos, nos dara error, pero es porque aun no hemos definido su funcion en el controlador:

```js
public function update(Request $request, $id)
{
    $product = Product::findOrFail($id);


    $product->update([
        'name' => $request->input('name')
    ]);
}
```


**Validaciones**

Si queremos validar algunos campos recibidos, escribiremos la sentencia validate en nuestros metodos.


```js
public function store(Request $request, $id)
{
    $this->validate($request, [
        'name' => 'required|unique:products'
    ]);

    return Product::create([
        'name' => $request->input('name')
    ]);
}
```

**Testeo (Parte 3)**

Añadiremos las siguientes validaciones para verificar en caso de que el campo name este vacio y en caso de haber nombre duplicado:


```js
public function testProductCreationFailsWhenNameNotProvided()
{
    $product = factory(\App\Product::class)->make(['name'=> '']);

    $this->post(route('api.products.store'), $product->jsonSerialize(), $this->jsonHeaders())
        ->seeJson(['name'=> ['The name field is required']])
        ->assertResponseStatus(422);
}

public function testProductCreationFailsWhenNameNotUnique()
{
    $name = 'feets';
    $product1 = factory(\App\Product::class)->create(['name'=> $name]);
    $product2 = factory(\App\Product::class)->make(['name'=> $name]);

    $this->post(route('api.products.store'), $product->jsonSerialize(), $this->jsonHeaders())
        ->seeJson(['name'=> ['The name has already been taken']])
        ->assertResponseStatus(422);
}
```

**Service Providers**

Para ejecutar proveedores nos vamos a la carpeta app/providers, en ella hay archivos como AppServiceProvider en el que podemos crear validaciones que queremos ejecutar en cualquier parte del proyecto.

En este caso verificaremos que es una palabra y de que termine en algunos sufijos determinados:

```js
use Illuminate\Support\Facades\Validator;

public function boot()
{
    Validator::extend('productQuality', function($attribute, $value, $parameters){
        return !preg_match('/[^A-Za-z]/', $value) && preg_match('/eets|eats|uites|etes|ites|ettes$/', value);
    });
}
```

Ahora como hemos agregado una nueva validacion, debe tener un mensaje de error que sea entendible, para eso vamos a la carpeta resources/lang/en/validation.php

Aqui agregaremos nuestra nueva validacion y le añadiremos un mensaje de error en caso de que suceda, asi (debes tener en cuenta el snake_case):

` 'product_quality'  => 'The product name provided not match our standard of excellence',
`

Entonces para cuando estemos escribiendo nuestras reglas de validacion, tambien tendremos que escribir la que acabamos de crear

















Fin

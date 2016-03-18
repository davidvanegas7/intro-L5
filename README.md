
Primeros pasos Laravel 5
------------------------

**Creacion del proyecto**

Laravel 5 crea el proyecto con el comando:

`composer create-project laravel/laravel --prefer-dist 5.* nombreproyecto
`

Entre sus diferencias mas notorias esta el hecho de que (al igual que symfony2) crea un archivo env donde estan los datos de tu base de datos y tu gestor de emails.

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

-------------------------------------
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

-------------------------------------
**Rutas**

La parte de rutas queda alojada en la direccion app/Http/routes.php

Para hacer pruebas sin tener que configurar un servidor en el momento, utilizamos el comando:

`php artisan serve --host=192.168.64.166
`

O la ip que tu quieras ponerle.

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

----------------------------------
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





Fin

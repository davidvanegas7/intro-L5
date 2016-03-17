
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

























Fin

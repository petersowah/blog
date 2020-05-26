---
layout: post
title:  "Using UUIDs with Spatie Laravel Permission Package"
description: "A blog post on using Spatie's Roles and Permissions library with UUID primary keys"
date: 2019-04-05
feature_image: /images/posts/2019/laravel-uuid.jpeg 
tags: [laravel, php]
---

The [Laravel Permissions](https://github.com/spatie/laravel-permission){:target="_blank"} package developed by one of my favourite people in the World, the team at [Spatie](https://spatie.be){:target="_blank"}. This package easily allows you to add the functionality of an Access Control List(ACL) to your Laravel project. You can define roles for users, then define as many permissions as you wish for these roles or directly to a user.

<!--more-->

The package ships with support for auto-incrementing primary keys as the default, just as is with the Laravel framework. You can swap this out for UUID based primary keys for your database tables.

## What are UUIDs?
UUID is an abbreviation for "Universal Unique Identifier". It is an alpha-numeric string that is 36 characters long(32 hexadecimal characters separated by four hyphens). It's a 128-bit value used for a unique identification. It is also known as the GUID (Globally Unique Identification), especially in the world of Microsoft.

## Benefits of using UUIDs as PKs against auto-incrementing IDs
1. It's easier to shard.
2. It's easier to merge/replicate. There is no universal order, hence easy merging of records from different databases.
3. You know the ID before the insert, it can simplify the logic/flow.
4. They are very unique across applications - every table, every database, every server.
5. You can generate IDs anywhere, instead of having to roundtrip to the database server. It also simplifies logic in the application. For example, to insert data into a parent table and child tables, you have to insert into the parent table first, get generated id and then insert data into the child tables. By using UUID, you can generate the primary key value of the parent table up front and insert rows into both parent and child tables at the same time within a transaction.
6. UUID values do not expose the information about your data so they are safer to use in a URL. For example, if a customer with id 10 accesses his account via http://www.foobar.com/customers/69/ URL, it is easy to guess that there is a customer 70, 71, etc., and this could be a target for an attack.

## Downsides of using UUIDs as PKs against auto-incrementing IDs
1. There is a huge overhead in size. A UUID/GUID is four times larger than an INT. This is very important when it comes to indexes.
2. Can't order by ID to get the insert order. This is a minor one as this can be achieved by sorting timestamps on records eg. created_at
3. Debugging seems to be  more difficult, imagine the expression WHERE id = '9eccf14d-2242-4018-bdc8-e648d4af7611;' instead of WHERE id = 1

## Setting up Laravel Permissions
Spatie usually provides good,very detailed documentation for their packages. You can find documentation for the Laravel Permission package [here](https://github.com/spatie/laravel-permission#installation){:target="_blank"}.

First, we install the package using composer:  
`composer require spatie/laravel-permission`  

When the installation of the package is done, the service provider is registered automatically. On the other hand, we can manually register the service provider in our `config/app/php` file:  
{% highlight php %}
'providers' => [
    // ...
    Spatie\Permission\PermissionServiceProvider::class,
];
{% endhighlight php %}

Next, we publish the migration for the tables with:  
`php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"`  

After publishing the migration, let's run:  
`php artisan migrate`  

then publish the config file with:  
`php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"`  

This command publishes a config file for the package in `config/permission.php`.

Great! We are done setting up the package in project.

Now let's setup our Eloquent models to use Uuid as the primary key of database tables.

We are going to take advantage of the events in Laravel. To be more specific, we are going to use the ‘_creating_’ event. This event is triggered exactly when a new record of the model is getting created.

One of the cleaner ways is to, extract it into a trait by itself and do all the logic there then you can just use that trait when you need a model to use UUIDs.
{% highlight php %}
namespace App;

use Illuminate\Support\Str;

trait Uuid
{
    /**
    * Boot function from Laravel
    */
    protected static function boot()
    {
        parent::boot();
        static::creating(function ($model) {
            $model->incrementing = false;
            $model->keyType = 'string';
            $model->{$model->getKeyName()} = Str::uuid()->toString();
        });
    }
}
{% endhighlight %}

From the above snippet, we are hooking into the boot method of the model and when the _creating_ model is called, we pass in a closure. We are setting setting autoincrementing of the off(_setting to false_). The method ‘getKeyName’ will get the name of the primary key, just in case you are using something other than the  ‘id’ for the primary key.

Now that we have our trait, we can use it in all of our models(_eg. User model_) like so:

{% highlight php%}
namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Authenticatable
{
    use Notifiable;
    use HasRoles;
    use SoftDeletes;
    use Uuid;
    
    protected $primaryKey = 'id';
{% endhighlight php%}

Great! So now all of our models have have uuid as the format for primary keys. Let's set up a corresponding database migration for our User model.
{% highlight php %}
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('first_name');
            $table->string('last_name');
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('phone');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('users');
    }
}
{% endhighlight %}
As you can see, we are making use of Laravel's uuid column type for our _id_ field and indexing it as the primary key.

Let's setup a User Factory.
{% highlight php%}
use Illuminate\Support\Str;
use Faker\Generator as Faker;

$factory->define(App\User::class, function (Faker $faker) {
    return [
        'first_name' => $faker->firstName,
        'last_name' => $faker->lastName,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'phone' => $faker->phoneNumber,
        'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
        'remember_token' => Str::random(10),
    ];
});
{% endhighlight php %}
Let's fire up Tinker;  
`php artisan tinker`  

We create a user by running the following command;
{% highlight php %}
$user = factory(App\User::class)->create();
{% endhighlight %}

This creates a user object like this;
```
{
  “first_name”: “Sheila”,
  "last_name": "Ayivor",
  ”email”: “foo@bar.com”,
  ”id”: “449c133a-2790-44be-a492-65a20094f392”,
  ”updated_at”: “2019–04–02 21:30:50”,
  ”created_at”: “2019–04–02 21:30:50”
}
```

Now we are sure that everything works just as we want it to. There is a problem though, the tables generated by the Laravel Permission package will have ids as the primary key. According to the documentation, we need to do the below;
>"If you're using UUIDs or GUIDs for your User models you can update the `create_permission_tables.php` migration and replace `$table->unsignedBigInteger($columnNames['model_morph_key'])` with `$table->uuid($columnNames['model_morph_key'])`. For consistency, you can also update the package configuration file to use the `model_uuid` column name instead of the default `model_id` column."
Following this instruction, we end up with a create permission table like this;
{% highlight php %}
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePermissionTables extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        $tableNames = config('permission.table_names');
        $columnNames = config('permission.column_names');

        Schema::create($tableNames['permissions'], function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('name');
            $table->string('guard_name');
            $table->timestamps();
        });

        Schema::create($tableNames['roles'], function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('name');
            $table->string('guard_name');
            $table->timestamps();
        });

        Schema::create($tableNames['model_has_permissions'], function (Blueprint $table) use ($tableNames, $columnNames) {
            $table->uuid('permission_id');

            $table->string('model_type');
            $table->uuid($columnNames['model_morph_key']);
            $table->index([$columnNames['model_morph_key'], 'model_type', ]);

            $table->foreign('permission_id')
                ->references('id')
                ->on($tableNames['permissions'])
                ->onDelete('cascade');

            $table->primary(
                ['permission_id', $columnNames['model_morph_key'], 'model_type'],
                'model_has_permissions_permission_model_type_primary'
            );
        });

        Schema::create($tableNames['model_has_roles'], function (Blueprint $table) use ($tableNames, $columnNames) {
            $table->uuid('role_id');

            $table->string('model_type');
            $table->uuid($columnNames['model_morph_key']);
            $table->index([$columnNames['model_morph_key'], 'model_type', ]);

            $table->foreign('role_id')
                ->references('id')
                ->on($tableNames['roles'])
                ->onDelete('cascade');

            $table->primary(
                ['role_id', $columnNames['model_morph_key'], 'model_type'],
                'model_has_roles_role_model_type_primary'
            );
        });

        Schema::create($tableNames['role_has_permissions'], function (Blueprint $table) use ($tableNames) {
            $table->uuid('permission_id');
            $table->uuid('role_id');

            $table->foreign('permission_id')
                ->references('id')
                ->on($tableNames['permissions'])
                ->onDelete('cascade');

            $table->foreign('role_id')
                ->references('id')
                ->on($tableNames['roles'])
                ->onDelete('cascade');

            $table->primary(['permission_id', 'role_id']);
        });

        app('cache')
            ->store(config('permission.cache.store') != 'default' ? config('permission.cache.store') : null)
            ->forget(config('permission.cache.key'));
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        $tableNames = config('permission.table_names');

        Schema::drop($tableNames['role_has_permissions']);
        Schema::drop($tableNames['model_has_roles']);
        Schema::drop($tableNames['model_has_permissions']);
        Schema::drop($tableNames['roles']);
        Schema::drop($tableNames['permissions']);
    }
}
{% endhighlight %}

This solves our problem right? Now we have our roles and permissions tables having uuid primary keys. In my case not so much. I still ran into issues where Eloquent was expecting an integer as primary key even after running `composer dumpautoload`. If that's the case with you as well, please continue reading.

To resolve this, we create two models `Role.php` and `Permission.php`.
{% highlight php %}
namespace App;

use Illuminate\Database\Eloquent\Model;
use Spatie\Permission\Models\Role as SpatieRole;

class Role extends SpatieRole
{
    use Uuid;
    protected $primaryKey = 'id';
    public $incrementing = false;

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'id' => 'string'
    ];
}
{% endhighlight %}
and 
{% highlight php %}
namespace App;

use Illuminate\Database\Eloquent\Model;
use Spatie\Permission\Models\Permission as SpatiePermission;

class Role extends SpatiePermission
{
    use Uuid;
    protected $primaryKey = 'id';
    public $incrementing = false;

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'id' => 'string'
    ];
}
{% endhighlight %}
As you can see, we are extending the models from the package(_Spatie\Permission\Models\Permission_ and _Spatie\Permission\Models\Role_). We are also using our trait in these classes without modifying the original class in the package.

Run `composer dumpautoload`.  

Try creating a role, permission, assign permission to a role and assign roles to users. You shouldn't run into any issues.

Cheers and thanks for reading!
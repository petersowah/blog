---
layout: post
title:  "Black Panther REST API with Lumen and Cloudinary"
date: 2018-02-23
tags: [lumen, php, api, cloudinary]
---
{% include image_full.html imageurl="/assets/images/posts/2018/black-panther.jpeg" title="Black Panther Banner" caption="Black Panther Banner" %}

About a week ago(15th February, 2018), the first Black Panther movie was premiered in cinemas around the world. Woop! Woop! I am a huge Marvels movies(and some Sci-fi) and a keen follower of the MCU(Marvel Cinematic Universe). Let’s go ahead and write an API that returns data on Black Panther characters. Let’s make Shuri proud! Don’t Freeze! 😃
<!--more-->

## What is Lumen?
>Lumen is a stunningly fast PHP micro-framework for building web applications with expressive, elegant syntax. We believe development must be an enjoyable, creative experience to be truly fulfilling. Lumen attempts to take the pain out of development by easing common tasks used in the majority of web projects, such as routing, database abstraction, queueing, and caching. <cite>[Github](https://github.com/laravel/lumen){:target="_blank"}</cite>

## Project Requirements
Lumen, like any other [framework](https://github.com/showcases/web-application-frameworks){:target="_blank"}, has system requirements that need to be met for a smooth development process and running of the application. Requirements of Lumen include;

1. PHP ≥ 7.1.3 (including OpenSSL, PDO and Mbstring PHP extensions must be installed)
2. [Composer](https://getcomposer.org){:target="_blank"}
3. A MySQL database([instructions on setup](https://dev.mysql.com/doc/refman/5.7/en/installing.html){:target="_blank"})
4. A database tool for MySQL(Choose from [here](https://medium.com/bestoflist/20-best-mysql-gui-tools-2017-e1abe055475e){:target="_blank"}) — For managing our MySQL database. I use [SequelPro](https://www.sequelpro.com/){:target="_blank"} because I am on a Mac.

## Getting Started
Let’s dig in. We first pull in the Lumen installer through [Composer](https://getcomposer.org){:target="_blank"} — a PHP Dependency Manager. If you don’t already have Composer, get it [here](https://getcomposer.org/download/){:target="_blank"} and install it globally. We then run the following command to install Lumen on our system. Find instructions on setting up Lumen [here](https://lumen.laravel.com/docs/5.6#installing-lumen){:target="_blank"}.

With our Lumen setup complete, we create a new project. The command for creating a new Lumen project is;  
`lumen new [app name]`

In our case, we run:  
`lumen new black-panther-api`

This command sets up a fresh lumen project in a folder called _black-panther-api_. We **cd** into the directory:  
`cd black-panther-api`

**PS**: To be sure all packages are installed, let’s run composer install to repeat the process(pull in packages). Next, we boot up the application with the following command:  
`php -S localhost:8000 -t public`

This serves our application. We can access it by going to http://localhost:8000 in our browser. You should see the version of Lumen installed returned or printed out on your screen. In this case; Lumen (5.6.1) (Laravel Components 5.6.*) -the latest version as of time writing this blog post.

## Getting Our Hands Dirty

Great! So far so good. (This is where you take a sip from your cup of coffee). We are going to leverage on [Eloquent](https://laravel.com/docs/5.6/eloquent) - Laravel’s sassy ORM and Facades(understand the Facade Design Pattern [here](https://www.sitepoint.com/how-laravel-facades-work-and-how-to-use-them-elsewhere/)). These are ‘deactivated’ by default in Lumen. We ‘activate’ them by uncommenting two lines of code in the bootstrap/app.php file on lines 26 and 28 so we have;
{% highlight php%}
$app->withFacades();

$app->withEloquent();
{% endhighlight %}

Let’s change the text returned on hitting the base url(http://localhost:8000) of our application. Open the file web.php in the routes folder. Delete the line that has
{% highlight php%}
return $router->app->version();
{% endhighlight %}

replace with;
{% highlight php%}
return 'Black Panther API v1.0.0 - Wakanda Forever!';
{% endhighlight %}

Save the file and refresh or load http://localhost:8000 to see changes take effect.

## Database — Models and Migrations
Lumen makes connecting to databases and running queries extremely simple. Currently Lumen supports four database systems: MySQL, Postgres, SQLite, and SQL Server.

The [Lumen documentation](https://lumen.laravel.com/docs/5.6){:target="_blank"} defines, Migrations as follows;

>Migrations are like version control for your database, allowing your team to easily modify and share the application’s database schema.<cite>[Lumen Docs](https://lumen.laravel.com/docs/5.6){:target="_blank"}</cite>

Models on the other hand, represent knowledge — the underlying, logical structure of data in a software application and the high-level class associated with it. A model could be a single object (rather uninteresting), or it could be some structure of objects. This object model does not contain any information about the user interface.

Let’s create our migration file to create the Characters table:  
`php artisan make:migration create_characters_table`

This command creates a file in the **_database/migrations_** directory. All migrations are timestamped and this is how the application keeps track of migrations in order. In my case, **_2018_01_24_133302_create_characters_table.php_**. We open this file in our favorite IDE and then add the attributes of Black Panther characters in there. We want our characters to have the following attributes: id, uuid, name, alias, occupation, gender, place of birth, bio, abilities, played by, image_path which will create corresponding columns in our database.

Our migration file with the above attributes will look like this:  
{% highlight php%}
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCharactersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('characters', function (Blueprint $table) {
            $table->increments('id');
            $table->uuid('uuid')->unique(); 
            $table->string('name')->unique();
            $table->string('alias');
            $table->string('occupation');
            $table->string('gender');
            $table->string('place_of_birth');
            $table->text('bio');
            $table->text('abilities');
            $table->string('played_by');
            $table->string('image_path');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('characters');
    }
}
{% endhighlight %}

We then run our migration;  
{% highlight php%}
php artisan migrate
{% endhighlight %}

This creates a characters table together with the columns defined in our database. We should have two tables in our database now; **characters** and **migrations**.

Next step is the creation of the model. Create the **app/Character.php** file and add the following code:
{% highlight php %}
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Character extends Model
{
   public $timestamps = false;
   protected $primaryKey = 'uuid';
   /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
   protected $fillable = [
      'uuid', 'name', 'occupation', 'place_of_birth', 'gender', 'abilities', 'played_by', 'image_path', 'bio', 'alias'
   ];

   /**
    * The attributes excluded from the model's JSON form.
    *
    * @var array
    */
   protected $hidden = ['id'];
}
{% endhighlight %}

In the above code, we have made the **id** attribute hidden from the JSON responses returned. You can read on mass assignment here.

## Setting Up Cloudinary
Cloudinary is the media management platform for web and mobile developers. It enables users to upload, store, manage, manipulate and deliver images and video for websites and apps, with the goal of improving performance.

Head over to [Cloudinary](https://cloudinary.com/), to sign up for an account. If you already have an account, just sign in. Once done, you get presented with a dashboard like this;
{% include image_caption.html imageurl="/assets/images/posts/2018/cloudinary-dashboard.png" title="Cloudinary Dashboard" caption="Cloudinary Dashboard" %}


From the dashboard, we are able to see our account details. We are going to need the Cloudinary cloud name, API Key and API secret. These values will be stored in the .env file in the root of the project. Replace ‘****’ with the corresponding values from your Cloudinary dashboard from below.

{% highlight php%}
CLOUDINARY_API_KEY=******************
CLOUDINARY_API_SECRET=******************
CLOUDINARY_CLOUD_NAME=******************
{% endhighlight %}

Let’s pull in the Cloudinary package from [packagist](https://packagist.org){:target="_blank"}. We search for Cloudinary and click on cloudinary/cloudinary_php from the list of results. We run the following command pull in the Cloudinary package into our project through Composer.
{% highlight php%}
composer require cloudinary/cloudinary_php
{% endhighlight %}

To set Cloudinary configuration values at runtime, we pass an array to the config method in the Cloudinary class. We do this in the bootstrap/app.php file. See below.
{% highlight php%}
Cloudinary::config(array(
   'cloud_name' => env('CLOUDINARY_CLOUD_NAME', true),
   'api_key' => env('CLOUDINARY_API_KEY', true),
   'api_secret' => env('CLOUDINARY_API_SECRET', true)
));
{% endhighlight %}

## Routing — The Border Tribe
You will define all of the routes for your application in the `routes/web.php` file. Open this file in your IDE and the code below.
{% highlight php%}
<?php

/*
|--------------------------------------------------------------------------
| Application Routes
|--------------------------------------------------------------------------
|
| Here is where you can register all of the routes for an application.
| It is a breeze. Simply tell Lumen the URIs it should respond to
| and give it the Closure to call when that URI is requested.
|
*/

$router->get('/', function () use ($router) {
   return 'Black Panther v1.0.0 - Wakanda Forever!';
});

$router->group(['prefix' => 'api/v1'], function() use ($router){
   $router->get('characters', 'CharacterController@index');
   $router->post('characters', 'CharacterController@create');
   $router->get('characters/{uuid}', 'CharacterController@show');
   $router->put('characters/{uuid}', 'CharacterController@update');
   $router->delete('characters/{uuid}', 'CharacterController@destroy');
});
{% endhighlight %}

From the code in the **web.php** file, we leverage on [routes](https://lumen.laravel.com/docs/5.6/routing){:target="_blank"} and the **CharacterController**(which we will create next). Route groups allow us to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. Find out more on routes [here](https://lumen.laravel.com/docs/5.6/routing){:target="_blank"}.

## The Controller
Instead of defining all of your request handling logic in a single `routes.php` file, you may wish to organize this behavior using Controller classes. Controllers can group related HTTP request handling logic into a class. Controllers are stored in the `app/Http/Controllers` directory.

We create our controller, CharacterController in `app/Http/Controllers`. We then add the following code:
{% highlight php%}
<?php

namespace App\Http\Controllers;

use App\Character;
use Illuminate\Http\Request;
use Ramsey\Uuid\Uuid;

use Cloudinary\Uploader;

class CharacterController extends Controller
{
   public $timestamps = false;
   /**
    * Create a new controller instance.
    *
    * @return void
    */
   public function __construct()
   {
      //
   }

   public function index(){
      return response()->json(['data' => Character::all()]);
   }

   public function create(Request $request){
      $uuid = Uuid::uuid4();

      $this->validate($request, [
         'name' => 'required|unique:characters',
         'occupation' => 'required',
         'gender' => 'required',
         'place_of_birth' => 'required',
         'played_by' => 'required',
         'image_path' => 'required',
         'bio' => 'required',
      ]);

      $extension = $request->image_path->extension();

      $image = Uploader::upload($request->file('image_path')->getRealPath(), ['public_id' => strtolower($request->input('name'). '.' .$extension), 'folder' => 'black-panther']);

      $character = Character::create(array_merge($request->all(), [ 'uuid' => $uuid->toString(), 'image_path' => $image['secure_url'] ]));

      return response()->json(['data' => $character, 'status' => 201]);
   }

   public function show($uuid){
      $character = Character::where('uuid', $uuid)->firstOrFail();

      return response()->json(['data' => $character ]);
   }

   public function update(Request $request, $uuid){
      $character = Character::where('uuid', $uuid)->firstOrFail();

      $character->update($request->all());

      return response()->json(['data' => $character, 'status' => 200]);
   }

   public function destroy($uuid){
      Character::where('uuid', $uuid)->firstOrFail()->delete();

      return response('Character Squashed', 200);
   }
}
{% endhighlight %}

These methods list, create, show, update and destroy/delete a resource.

+ The **index** method handles a **_GET_** request to the characters endpoint and returns all Black Panther characters resources in the database.

+ The **create** method handles a **_POST_** request to the characters endpoint and creates a new character resource. We also upload the character’s avatar to the cloud(Cloudinary servers) programmatically using their API. On successful upload, an object with information on file is returned and is stored in image_path variable. You can find out more in the official documentation. We store the path(secure url) to the avatar in our database.

+ The **show** method also handles a **_GET_** request to the characters endpoint with a specified resource. Eg. _/characters/10a870d8-bdb9–4856–8866-c491008fd921_ and returns the resource with the **uuid** specified.

+ The **update** method checks if a specified character resource exists and allows the resource to be updated. This method handles a PUT request to the characters endpoint and updates specified resource. Eg. _/characters/10a870d8-bdb9–4856–8866-c491008fd921_

+ The **destroy** method also checks if a specified resource characters resource exists and then deletes it from the database. This method handles a DELETE request to the characters endpoint with a character resource specified.

We realize from our routes file(web.php), that the HTTP request being sent to a particular endpoint, determines which controller action/method is invoked. Looking through the code, we import or require the Character model and the UUID class via:  
{% highlight php%}
use App\Author;
use Ramsey\Uuid\Uuid;
{% endhighlight %}

In the controller, all methods return data in the JSON format.

## Testing [Manual]
We cab test our API using REST clients(I recommend either [Insomnia](https://insomnia.rest/){:target="_blank"} or [PostMan](https://www.getpostman.com/){:target="_blank"}). Check out various documentations on testing using either tool.

## Conclusion
We did it! We successfully wrote a REST API listing characters in Black Panther using Lumen(super-cool micro-framework) and Cloudinary. Lumen is my go-to tool for APIs using PHP. I hope you get to enjoy using it as much as I do. You can find the source for this project on [github](https://github.com/petersowah/black-panther-api){:target="_blank"}. Cheers!

_Please let me know if you have any questions, observations or criticisms in the comment section. Feedback is always welcome._
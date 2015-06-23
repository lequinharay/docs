# HTTP Controllers

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
- [Controller Middleware](#controller-middleware)
- [RESTful Resource Controllers](#restful-resource-controllers)
	- [Partial Resource Routes](#restful-partial-resource-routes)
	- [Naming Resource Routes](#restful-naming-resource-routes)
	- [Nested Resources](#restful-nested-resources)
	- [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
- [Implicit Controllers](#implicit-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Introduction

routes.phpの1ファイル内にすべてのリクエストのハンドリングロジックをすべて定義する代わりに、Controllerクラスを使って振る舞いを構成することができます。コントローラーを使うと、関連するHTTPリクエストのハンドリングロジッククラスとしてグループ化することができます。また、コントローラーは原則的には`app/Http/Controllers`ディレクトリに格納されます。

<a name="basic-controllers"></a>
## Basic Controllers

ここでは基本的なコントローラークラスの例を示します。すべてのLaravelコントローラーはデフォルトのLaravelによってインストールされる基底コントローラークラスを継承しなければなりません。


	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * 指定したユーザーのプロフィールを表示する
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			return view('user.profile', ['user' => User::findOrFail($id)]);
		}
	}


以下の様に書けば、コントローラーへのルーティングが可能です。

	Route::get('user/{id}', 'UserController@showProfile');

リクエストがここで指定されているルートURIにマッチしたら、`UserController`クラスの`showProfile`メソッドが実行されます。もちろんルートパラメーターはメソッドに渡されます。

#### Controllers & Namespaces

これは大変重要なことですが、コントローラーのルーティングを定義する際に、コントローラーの完全な名前空間を特定する必要はないということに注意する必要があります。`App\Http\Controllers`のあとにくるクラス名の一部のみを定義することになります。デフォルトでは、`RouteServiceProvider`はroutes.phpファイルをルートコントローラーの名前空間を含むルーティンググループも一緒にロードします。

`App\Http\Controllers`ディレクトリよりも深い階層のPHP名前空間を使ってコントローラをネストしたり構成したりする場合は、単純に`App\Http\Controllers`以降の相対指定を使ってください。例えばコントローラーのフルパスが`App\Http\Controllers\Photos\AdminController`である場合は、以下のようにルーティングを登録することになります。

	Route::get('foo', 'Photos\AdminController@method');

#### Naming Controller Routes

クロージャーのルーティングのように、コントローラーのルーティングを指定することもできます。

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

一度コントローラーのルーティングに名前を割り当てると、このアクションへのURLを簡単に生成できるようになります。コントローラーアクションへのURLを生成するには、`action`ヘルパーメソッドを使います。登録後は、`App\Http\Controllers`以降のコントローラークラス名の部分を指定するだけで良くなります。

	$url = action('FooController@method');

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) はコントローラーのルーティングを以下のように割り当てます。

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

ただし、コントローラーのコンストラクタ内でミドルウェアを特定してしまうほうがより簡便です。`middleware`メソッドをコントローラーのコンストラクタ内で使うと、ミドルウェアを簡単にコントローラーに割り当てられます。さらに、コントローラークラス内で、特定のメソッドのみに割り当てるような制限をかけることもできます。

	class UserController extends Controller
	{
		/**
		 * Instantiate a new UserController instance.
		 *
		 * @return void
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}
	}

<a name="restful-resource-controllers"></a>
## RESTful Resource Controllers

Resource controllers make it painless to build RESTful controllers around resources. For example, you may wish to create a controller that handles HTTP requests regarding "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

	php artisan make:controller PhotoController

The Artisan command will generate a controller file at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.

Next, you may register a resourceful route to the controller:

	Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have methods stubbed for each of these actions, including notes informing you which URIs and verbs they handle.

#### Actions Handled By Resource Controller

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

<a name="restful-partial-resource-routes"></a>
#### Partial Resource Routes

When declaring a resource route, you may specify a subset of actions to handle on the route:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### Naming Resource Routes

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### Nested Resources

Sometimes you may need to define routes to a "nested" resource. For example, a photo resource may have multiple "comments" that may be attached to the photo. To "nest" resource controllers, use "dot" notation in your route declaration:

	Route::resource('photos.comments', 'PhotoCommentController');

This route will register a "nested" resource that may be accessed with URLs like the following: `photos/{photos}/comments/{comments}`.

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class PhotoCommentController extends Controller {

		/**
		 * Show the specified photo comment.
		 *
		 * @param  int  $photoId
		 * @param  int  $commentId
		 * @return Response
		 */
		public function show($photoId, $commentId)
		{
			//
		}

	}

<a name="restful-supplementing-resource-controllers"></a>
#### Supplementing Resource Controllers

If it becomes necessary to add additional routes to a resource controller beyond the default resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:

	Route::get('photos/popular', 'PhotoController@method');

	Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## Implicit Controllers

Laravel allows you to easily define a single route to handle every action in a controller class. First, define the route using the `Route::controller` method. The `controller` method accepts two arguments. The first is the base URI the controller handles, while the second is the class name of the controller:

	Route::controller('users', 'UserController');

 Next, just add methods to your controller. The method names should begin with the HTTP verb they respond to followed by the title case version of the URI:

 	<?php namespace App\Http\Controllers;

	class UserController extends Controller
	{
		/**
		 * Responds to requests to GET /users
		 */
		public function getIndex()
		{
			//
		}

		/**
		 * Responds to requests to GET /users/show/1
		 */
		public function getShow($id)
		{
			//
		}

		/**
		 * Responds to requests to GET /users/admin-profile
		 */
		public function getAdminProfile()
		{
			//
		}

		/**
		 * Responds to requests to POST /users/profile
		 */
		public function postProfile()
		{
			//
		}
	}

As you can see in the example above, `index` methods will respond to the root URI handled by the controller, which, in this case, is `users`.

#### Assigning Route Names

If you would like to [name](/docs/{{version}}/routing#named-routes) some of the routes on the controller, you may pass an array of names as the third argument to the `controller` method:

	Route::controller('users', 'UserController', [
		'getShow' => 'user.show',
	]);

<a name="dependency-injection-and-controllers"></a>
## 依存性注入とコントローラー

#### コンストラクタインジェクション

Laravelの[サービスコンテナ](/docs/{{version}}/container)はすべてのLaravelコントローラーの解決に使われています。結果として、コンストラクタ内でコントローラーが必要とする依存性をタイプヒンティングできるようになります。この依存性は自動的に解決され、コントローラーインスタンスに注入されます。


	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller
	{
		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}
	}

当然、あらゆる[Laravel contract](/docs/{{version}}/contracts)もタイプヒンティングすることができます。

#### メソッドインジェクション

コンストラクタインジェクションに加え、コントローラーのアクションメソッドの依存性もタイプヒンティングできます。例えば`Illuminate\Http\Request`のインスタンスをメソッドのひとつに関してタイプヒンティングしてみましょう。

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}
	}
	
	
コントローラーメソッドがルーティングパラメーターからの入力も受け付ける場合、単純に依存性の後ろにルーティング引数を並べることで実現できます。


	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
		/**
		 * Update the specified user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}
	}

<a name="route-caching"></a>
## Route Caching

If your application is exclusively using controller based routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it take to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:

	php artisan route:cache

That's all there is to it! Your cached routes file will now be used instead of your `app/Http/routes.php` file. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you may wish to only run the `route:cache` command during your project's deployment.

To remove the cached routes file without generating a new cache, use the `route:clear` command:

	php artisan route:clear

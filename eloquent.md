# Eloquent: Getting Started

- [イントロダクション](#introduction)
- [モデルの定義](#defining-models)
	- [Eloquentモデルの規約](#eloquent-model-conventions)
- [複数のモデルを取得する](#retrieving-multiple-models)
- [単一のモデルを取得する / 総計する](#retrieving-single-models)
	- [総計を取得する](#retrieving-aggregates)
- [モデルを挿入 & 更新する](#inserting-and-updating-models)
	- [通常の追加](#basic-inserts)
	- [通常の更新](#basic-updates)
	- [まとめての割り当て](#mass-assignment)
- [モデルの削除](#deleting-models)
	- [Soft Deleting](#soft-deleting)
	- [Querying Soft Deleted Models](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes)
- [Events](#events)

<a name="introduction"></a>
## Introduction

LaravelのEloquent ORMはデータベースに作用する美しくシンプルなActiveRecordの実装です。それぞれのデータベーステーブルは、それに対応するテーブルとの相互作用に使われる"モデル"を持っています。モデルを使うことでテーブルのデータに対してクエリを発行することができるし、同様にテーブルに新しいデータを挿入することもできます。

Eloquentに取り組む前に、`config/database.php`のデータベース接続設定がされていることを確認して下さい。データベース設定に関するこれ以上の情報は、[the documentation](/docs/{{version}}/database#configuration)を参照してください。

<a name="defining-models"></a>
## モデルの定義

手始めに、Eloquentモデルを作ってみましょう。通常モデルは`app`ディレクトリ内にあります。ただし、`composer.json`ファイルにしたがってオートロードされる場所であればどこにでも配置することができます。すべてのEloquentモデルは`Illuminate\Database\Eloquent\Model`クラスを継承しています。

モデルインスタンスを作る最も簡単な方法は、`make:model`（[Artisanコマンド](/docs/{{version}}/artisan)）を使うことです。

	php artisan make:model User

もしモデルを生成するときにあなたが[データベースマイグレーション](/docs/{{version}}/schema#database-migrations)も生成したい場合は、`--migration`か`-m`オプションで指定することができます。

	php artisan make:model User --migration

	php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquentモデルの規約

ここでは、`Flight`モデルクラスの例を見てみましょう。このクラスは`flights`データベーステーブルから情報を取得したり、このテーブルに情報を保存したりします。

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
	    //
	}


#### テーブル名

`Flight`モデルの為にどのテーブルを使っているかをEloquentに教えなかったことに注意してください。明示的に指定されていない限りは、クラス名を"スネークケース"で複数形にしたものががテーブル名として使われていると解釈されます。したがってここでは、Eloquentは`Flight`モデルは`flights`テーブルにデータを保存することを想定します。モデルの`table`プロパティを定義することで、ルールに沿わないテーブルを指定することができます。

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The table associated with the model.
		 *
		 * @var string
		 */
		protected $table = 'my_flights';
	}

#### 主キー

Eloquent はそれぞれのテーブルが`id`という名前の主キーを持っていることを前提にしています。この規約は`$primaryKey`プロパティで定義することで上書きすることができます。

#### Timestamps

By default, Eloquent expects `created_at` and `updated_at` columns to exist on your tables.  If you do not wish to have these columns automatically managed by Eloquent, set the `$timestamps` property on your model to `false`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * Indicates if the model should be timestamped.
		 *
		 * @var bool
		 */
		public $timestamps = false;
	}

If you need to customize the format of your timestamps, set the `$dateFormat` property on your model. This property determines how date attributes are stored in the database, as well as their format when the model is serialized to an array or JSON:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The storage format of the model's date columns.
		 *
		 * @var string
		 */
		protected $dateFormat = 'U';
	}

<a name="retrieving-multiple-models"></a>
## 複数のモデルを取得する

一度モデルを作り、[データベースと関連付ける](/docs/{{version}}/schema)と、データベースからデータを取得する準備が整います。強力な[クエリビルダー](/docs/{{version}}/queries)としてのEloquentモデルによって、モデルと関連付けられたデータベーステーブルにクエリを投げられるようになります。例えば、

	<?php namespace App\Http\Controllers;

	use App\Flight;
	use App\Http\Controllers\Controller;

	class FlightController extends Controller
	{
		/**
		 * Show a list of all available flights.
		 *
		 * @return Response
		 */
		public function index()
		{
			$flights = Flight::all();

			return view('flight.index', ['flights' => $flights]);
		}
	}

#### Accessing Column Values

If you have an Eloquent model instance, you may access the column values of the model by accessing the corresponding property. For example, let's loop through each `Flight` instance returned by our query and echo the value of the `name` column:

	foreach ($flights as $flight) {
		echo $flight->name;
	}

#### 追加の制約を付加する

Eloquentの`all`メソッドはこのモデルのテーブルの結果をすべて返します。各々のEloquentモデルは[クエリビルダー](/docs/{{version}}/queries)を提供しているので、クエリに対して制約を付加し、`get`メソッドを使って結果を取得することが出来ます。

	$flights = App\Flight::where('active', 1)
				   ->orderBy('name', 'desc')
				   ->take(10)
				   ->get();

> **注意:** Eloquentモデルはクエリビルダーなので、[クエリビルダー](/docs/{{version}}/queries)の項の利用可能なすべてのメソッドを確認すべきです。Eloquentクエリ内で、これらのメソッドをすべて使うことが出来ます。

#### Collections

For Eloquent methods like `all` and `get` which retrieve multiple results, an instance of `Illuminate\Database\Eloquent\Collection` will be returned. The `Collection` class provides [a variety of helpful methods](/docs/{{version}}/eloquent-collections) for working with your Eloquent results. Of course, you may simply loop over this collection like an array:

	foreach ($flights as $flight) {
		echo $flight->name;
	}

#### 結果をchunkする

数千件のEloquentレコードを処理する必要がある場合、`chunk`コマンドを使います。この`chunk`メソッドはEloquentモデルの"chunk"を取得して、処理の`Closure`を提供してくれます。`chunk`メソッドを用いることで、巨大な結果セットを取り扱う時でもメモリを温存することができます。

	Flight::chunk(200, function ($flights) {
		foreach ($flights as $flight) {
			//
		}
	});

The first argument passed to the method is the number of records you wish to receive per "chunk". The Closure passed as the second argument will be called for each chunk that is retrieved from the database.

<a name="retrieving-single-models"></a>
## Retrieving Single Models / Aggregates

Of course, in addition to retrieving all of the records for a given table, you may also retrieve single records using `find` and `first`. Instead of returning a collection of models, these methods return a single model instance:

	// Retrieve a model by its primary key...
	$flight = App\Flight::find(1);

	// Retrieve the first model matching the query constraints...
	$flight = App\Flight::where('active', 1)->first();

#### Not Found Exceptions

Sometimes you may wish to throw an exception if a model is not found. This is particularly useful in routes or controllers. The `findOrFail` and `firstOrFail` methods will retrieve the first result of the query. However, if no result is found, a `Illuminate\Database\Eloquent\ModelNotFoundException` will be thrown:

	$model = App\Flight::findOrFail(1);

	$model = App\Flight::where('legs', '>', 100)->firstOrFail();

If the exception is not caught, a `404` HTTP response is automatically sent back to the user, so it is not necessary to write explicit checks to return `404` responses when using these methods:

	Route::get('/api/flights/{id}', function ($id) {
		return App\Flight::findOrFail($id);
	});

<a name="retrieving-aggregates"></a>
### Retrieving Aggregates

Of course, you may also use the query builder aggregate functions such as `count`, `sum`, `max`, and the other aggregate functions provided by the [query builder](/docs/{{version}}/queries). These methods return the appropriate scalar value instead of a full model instance:

	$count = App\Flight::where('active', 1)->count();

	$max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## モデルの挿入と更新

<a name="basic-inserts"></a>
### 基本的な挿入

データベースに新しいレコードを作るには、単純に新しいモデルインスタンスを作成し、モデルに属性値をセットし、`save`メソッドを呼びます。

	<?php namespace App\Http\Controllers;

	use App\Flight;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class FlightController extends Controller
	{
		/**
		 * Create a new flight instance.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			// Validate the request...

			$flight = new Flight;

			$flight->name = $request->name;

			$flight->save();
		}
	}

この例では、私達は単純にHTTPリクエストからきた`name`パラメーターを`App\Flight`モデルインスタンスの`name`属性に設定しています。`save`メソッドを呼ぶと、データベースにレコードが挿入されます。`save`メソッドが呼ばれると`created_at`と`updated_at`タイムススタンプが自動的にセットされるので、手動でこれらをセットする必要はありません。

<a name="basic-updates"></a>
### Basic Updates

The `save` method may also be used to update models that already exist in the database. To update a model, you should retrieve it, set any attributes you wish to update, and then call the `save` method. Again, the `updated_at` timestamp will automatically be updated, so there is no need to manually set its value:

	$flight = App\Flight::find(1);

	$flight->name = 'New Flight Name';

	$flight->save();

Updates can also be performed against any number of models that match a given query. In this example, all flights that are `active` and have a `destination` of `San Diego` will be marked as delayed:

	App\Flight::where('active', 1)
			  ->where('destination', 'San Diego')
			  ->update(['delayed' => 1]);

The `update` method expects an array of column and value pairs representing the columns that should be updated.

<a name="mass-assignment"></a>
### Mass Assignment

You may also use the `create` method to save a new model in a single line. The inserted model instance will be returned to you from the method. However, before doing so, you will need to specify either a `fillable` or `guarded` attribute on the model, as all Eloquent models protect against mass-assignment.

A mass-assignment vulnerability occurs when user's pass unexpected HTTP parameters through a request, and then that parameter changes a column in your database you did not expect. For example, a malicious user might send an `is_admin` parameter through an HTTP request, which is then mapped onto your model's `create` method, allowing the user to escalate themselves to an administrator.

So, to get started, you should define which model attributes you want to make mass assignable. You may do this using the `$fillable` property on the model. For example, let's make the `name` attribute of our `Flight` model mass assignable:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The attributes that are mass assignable.
		 *
		 * @var array
		 */
		protected $fillable = ['name'];
	}

Once we have made the attributes mass assignable, we can use the `create` method to insert a new record in the database. The `create` method returns the saved model instance:

	$flight = App\Flight::create(['name' => 'Flight 10']);

While `$fillable` serves as a "white list" of attributes that should be mass assignable, you may also choose to use `$guarded`. The `$guarded` property should contain an array of attributes that you do not want to be mass assignable. All other attributes not in the array will be mass assignable. So, `$guarded` functions like a "black list". Of course, you should use either `$fillable` or `$guarded` - not both:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Flight extends Model
	{
		/**
		 * The attributes that aren't mass assignable.
		 *
		 * @var array
		 */
		protected $guarded = ['price'];
	}

In the example above, all attributes **except for `price`** will be mass assignable.

#### その他の作成コマンド

複数の属性によってモデルを作成するには、他に2つのメソッドが存在します。`firstOrCreate`と`firstOrNew`です。`firstOrCreate`メソッドは与えられたカラムと値のペアを使ってデータベースレコードを配置します。モデルがデータベース内に見つからなかった場合、レコードは与えられた属性を使って挿入されます。

`firstOrNew`メソッドは`firstOrCreate`と似た様に、与えられた属性をデータベース内でマッチングしてレコードを配置します。ただしモデルがなかった場合には、新しいモデルインスタンスを返します。`firstOrNew`が返したモデルがまだデータベースとは同期されておらず、同期するためには手動で`save`メソッドを呼ぶ必要があります。

	// Retrieve the flight by the attributes, or create it if it doesn't exist...
	$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

	// Retrieve the flight by the attributes, or instantiate a new instance...
	$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## Deleting Models

To delete a model, call the `delete` method on a model instance:

	$flight = App\Flight::find(1);

	$flight->delete();

#### Deleting An Existing Model By Key

In the example above, we are retrieving the model from the database before calling the `delete` method. However, if you know the primary key of the model, you may delete the model without retrieving it. To do so, call the `destroy` method:

	App\Flight::destroy(1);

	App\Flight::destroy([1, 2, 3]);

	App\Flight::destroy(1, 2, 3);

#### Deleting Models By Query

Of course, you may also run a delete query on a set of models. In this example, we will delete all flights that are marked as inactive:

	$deletedRows = App\Flight::where('votes', '>', 100)->delete();

<a name="soft-deleting"></a>
### Soft Deleting

In addition to actually removing records from your database, Eloquent can also "soft delete" models. When models are soft deleted, they are not actually removed from your database. Instead, a `deleted_at` attribute is set on the model and inserted into the database. If a model has a non-null `deleted_at` value, the model has been soft deleted. To enable soft deletes for a model, use the `Illuminate\Database\Eloquent\SoftDeletes` trait on the model and add the `deleted_at` column to your `$dates` property:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;
	use Illuminate\Database\Eloquent\SoftDeletes;

	class Flight extends Model
	{
		use SoftDeletes;

		/**
		 * The attributes that should be mutated to dates.
		 *
		 * @var array
		 */
		protected $dates = ['deleted_at'];
	}

Of course, you should add the `deleted_at` column to your database table. The Laravel [schema builder](/docs/{{version}}/schema) contains a helper method to create this column:

	Schema::table('flights', function ($table) {
		$table->softDeletes();
	});

Now, when you call the `delete` method on the model, the `deleted_at` column will be set to the current date and time. And, when querying a model that uses soft deletes, the soft deleted models will automatically be excluded from all query results.

To determine if a given model instance has been soft deleted, use the `trashed` method:

	if ($flight->trashed()) {
		//
	}

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models

#### Including Soft Deleted Models

As noted above, soft deleted models will automatically be excluded from query results. However, you may force soft deleted models to appear in a result set using the `withTrashed` method on the query:

	$flights = App\Flight::withTrashed()
					->where('account_id', 1)
					->get();

The `withTrashed` method may also be used on a [relationship](/docs/{{version}}/eloquent-relationships) query:

	$flight->history()->withTrashed()->get();

#### Retrieving Only Soft Deleted Models

The `onlyTrashed` method will retrieve **only** soft deleted models:

	$flights = App\Flight::onlyTrashed()
					->where('airline_id', 1)
					->get();

#### Restoring Soft Deleted Models

Sometimes you may wish to "un-delete" a soft deleted model. To restore a soft deleted model into an active state, use the `restore` method on a model instance:

	$flight->restore();

You may also use the `restore` method in a query to quickly restore multiple models:

	App\Flight::withTrashed()
			->where('airline_id', 1)
			->restore();

Like the `withTrashed` method, the `restore` method may also be used on [relationships](/docs/{{version}}/eloquent-relationships):

	$flight->history()->restore();

#### Permanently Deleting Models

Sometimes you may need to truly remove a model from your database. To permanently remove a soft deleted model from the database, use the `forceDelete` method:

	// Force deleting a single model instance...
	$flight->forceDelete();

	// Force deleting all related models...
	$flight->history()->forceDelete();

<a name="query-scopes"></a>
## Query Scopes

Scopes allow you to define common sets of constraints that you may easily re-use throughout your application. For example, you may need to frequently retrieve all users that are considered "popular". To define a scope, simply prefix an Eloquent model method with `scope`:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Scope a query to only include popular users.
		 *
		 * @return \Illuminate\Database\Eloquent\Builder
		 */
		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		/**
		 * Scope a query to only include active users.
		 *
		 * @return \Illuminate\Database\Eloquent\Builder
		 */
		public function scopeActive($query)
		{
			return $query->where('active', 1);
		}
	}

#### Utilizing A Query Scope

Once the scope has been defined, you may call the scope methods when querying the model. However, you do not need to include the `scope` prefix when calling the method. You can even chain calls to various scopes, for example:

	$users = App\User::popular()->women()->orderBy('created_at')->get();

#### Dynamic Scopes

Sometimes you may wish to define a scope that accepts parameters. To get started, just add your additional parameters to your scope. Scope parameters should be defined after the `$query` argument:

	<?php namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Scope a query to only include users of a given type.
		 *
		 * @return \Illuminate\Database\Eloquent\Builder
		 */
		public function scopeOfType($query, $type)
		{
			return $query->where('type', $type);
		}
	}

Now, you may pass the parameters when calling the scope:

	$users = App\User::ofType('admin')->get();

<a name="events"></a>
## Events

Eloquent models fire several events, allowing you to hook into various points in the model's lifecycle using the following methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Events allow you to easily execute code each time a specific model class is saved or updated in the database.

<a name="basic-usage"></a>
### Basic Usage

Whenever a new model is saved for the first time, the `creating` and `created` events will fire. If a model already existed in the database and the `save` method is called, the `updating` / `updated` events will fire. However, in both cases, the `saving` / `saved` events will fire.

For example, let's define an Eloquent event listener in a [service provider](/docs/{{version}}/providers). Within our event listener, we will call the `isValid` method on the given model, and return `false` if the model is not valid. Returning `false` from an Eloquent event listener will cancel the `save` / `update` operation:

	<?php namespace App\Providers;

	use App\User;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * Bootstrap any application services.
	     *
	     * @return void
	     */
		public function boot()
		{
			User::creating(function ($user) {
				if ( ! $user->isValid()) {
					return false;
				}
			});
		}

		/**
		 * Register the service provider.
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}
	}

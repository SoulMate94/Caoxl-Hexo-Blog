---
title: Laravel 基础知识
date: 2018-01-16 15:06:24
categories: Laravel
tags: [Laravel, PHP, Framework]
---


> 记录Laravel的一些基础知识...
> > 原文地址: [Head into Laravel](http://blog.cjli.info/2016/08/19/Head-Into-Laravel/)

<!--more-->


# 准备

## 全局安装 `Composer`

```
    curl -sS https://getcomposer.org/installer | php -- --install-dir=bin --filename=composer
    
    # Or
    # php -r "readfile('https://getcomposer.org/installer');" | php
    
    # Or manually rename composer.phar to composer and link the composer binary file to $PATH
```

## 安装 `Laravel`/`Lumen`

```
    composer global require laravel/installer
    composer global require "laravel/lumen-installer=~1.0"
    
    vim ~/.zshrc
    
    # add `~/.composer/vendor/bin` to the $PATH
    # ...
    
    source ~/.zshrc
```

## 自带服务器运行

```
    laravel new site
    # or Lumen: `lumen new site`
    
    cd site
    php artisan serve --port 8888
```

## 注意

- 安装指定版本的 `Laravel/Lumen`

```
    composer create-project laravel/laravel blog "5.1.*"
```

# Blade

## 布局视图

```
    <!-- layouts/main.blade.php -->
    @yield('title')
    @yield('header')
    @yield('content')
    @yield('footer')
```

## 普通视图

```
    <!-- index.blade.php -->
    @extends('layouts.main')
    @section('title', 'PAGE NAME')
    @section('header')
      <!-- header detail -->
      {{ $variable->attr }}
    <!-- Include sub view -->
    @include('shared.navbar')
    @section('content')
      <!-- content detail -->
    	@if (...)
    		{{ time() }}
    	@elseif (...)
        	{{ some php code here }}
    	@else
          	{{ php }} or html all ok
      	@endif
          
    @section('footer')
      <!-- footer detail -->
    @stop
    <!-- or @endsection -->
```

更多例子参照([这里](https://cs.laravel-china.org/#blade), [这里](https://d.laravel-china.org/docs/5.1/blade))


# Data

## 将数据传递到视图

- 数组格式

```
    Route::get('/', function () {
    	
    	$data = [1, 2, 3] ;
        return view('welcome', ['data' => $data]);
    });
```

- `compact()`

```
    Route::get('/', function () {
    	
    	$data = [1, 2, 3] ;
        return view('welcome', compact('data'));
    });
```

- `with()`

```
    Route::get('/', function () {
    	
    	$data = [1, 2, 3] ;
        return view('welcome')->with('data' => $data);
    });
```

- 动态方法

```
    Route::get('/', function () {
    	
    	$data = [1, 2, 3] ;
        return view('welcome')->withData($data);
    });
```

## 数据库驱动

- `config/database.php` 

找到 `'default' => env('DB_CONNECTION', 'mysql')` ,修改成需要的驱动就可以了.

> If use sqlite(file based database for small project), please comment the DB_* fields in `.env` file.

## `Migration`

> **Migration: The Version Controll of Database**

```
    # 1. define the table we want to create
    php artisan make:migration create_users_table --create=user
    
    # 2. custom the table structure in /database/migrations/DATATIME_create_user_table.php@up() function
    
    # 3. actual create the table we defined before into our configured database
    php artisan migrate
    
    # 4. delete tables if we want modify it, migrate it to others env, etc.
    php artisan migrate:rollback
```

## `Tinker`

> 你的项目的所有PHP laravel方法能够被执行在Tinker

```
    php artisan tinker
    
    DB::table('users')->get();
```

命名空间等于文件夹结构。

## `命名空间`的回顾

在像PHP这样的编程语言中 命名空间用于放置类，避免类名相同时的歧义。

在laravel，定位的两种方法的类：

- use `use \Path\To\Class`; before the class defination.
- use `\Classname` in the function block.

**注意**，当您不使用任何命名空间时，PHP将在当前命名空间中搜索您需要的类。

## 查询生成器

这是`Laravel`一个传统与数据层交互方式。

按照这里的演示：[这里](https://cs.laravel-china.org/#db) AND [这里](http://d.laravel-china.org/docs/5.1/queries)

## `Eloquent`: ORM of Laravel

简单地说，它只是`MVC`设计模式中的`M`层。

不同的是，它是在PHP的活动记录实现与数据库表进行交互，而不是使用传统的查询生成器。

其结果是，每个表有一个模型，模型代表表和表的增删改查选项更方便和愉快的模型。

在`Laravel`中的使用步骤:

- 1.**Create table via `make:migration`**
- 2.**Insert data into the table**
- 3.**Make a Model reference to the table**

```
    php artisan make:model Models/User -m # --migration
```

`- m` 选项仅用于我们没有创建此模型响应的表时。

**注意**这里有一条规则：

> **Model name must be singular, but table name must be plurals.**

**模型名称必须是单数，但表名必须是复数。**

For example, model name of table `histories` should be `history`.

- 4.**Using Model in controller**

For example, we have created a table `users`, and a model `User` in namespace `App\Models\User`, contents of `App\Http\routes.php` like these:

在路由中使用:

```
    Route::get('users', 'Users@index');
    
    # below two routes are same intend, just for explain here
    # use only one way if you like
    
    // Will show specific user by id
    Route::get('users/{id}', 'Users@show');
    
    // Will show specific user by instance
    Route::get('users/{user}', 'Users@show');
```

在控制器中使用:

```
    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use App\Models\User;
    
    class Users extends Controller
    {
    	/**
    	 * get all users
    	 */
    	public function index()
    	{
    		return User::all() ;   // Automatically JSON Response
    	}
    	
    	
    	/**
    	 * 1. show specific user by id
    	 * !!! param name `$id` don't have to response to the `{id}` in routes defination
    	 */
        public function show($id)
        {
        	$user = User::find($id) ;    // Also JSON if has the result
        	return $user ? $user : 'user not found' ;
        }
        
        
        /**
    	 * 2. show specific user by instance
    	 * !!! param name `$user` must response to the `{user}` in routes defination
    	 */
        public function show(User $user)
        {
        	return $user ;    // JSON
        }
    }
```

- 5.**Pass the result data to blade views when necessary**

只需要注意: `.`是连接/目录的意思, 比如 `view(users.show)` 意思是 `resources/views/users/show.blade.php`


# `Eloquent`

举个简单的例子 我们创建表: `users`, `cards`, `notes`

```
    php artisan make:model Models\User -m
    php artisan make:model Models\Card -m
    php artisan make:model Models\Note -m
```

我们所定义的每个`up()`方法：

> 在 `database/migrations/`

- `CreateUsersTable:`

```
    Schema::create('users', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->timestamps();
            });
```

- `CreateCardsTable:`

```
    Schema::create('cards', function (Blueprint $table) {
                $table->increments('id');
                $table->integer('user_id')->unsigned()->index();
                $table->string('title');
                $table->timestamps();
            });
```

- `CreateNotesTable:`

```
    Schema::create('notes', function (Blueprint $table) {
                $table->increments('id');
                $table->integer('card_id')->unsigned()->index();
                $table->text('content');
                $table->timestamps();
            });
```

显然, 他们之间的关系: `user has cards`, `cards has notes`. 测试前, 运行迁移:

```
    php artisan migrate
```

Ok then. How to define the relationship with Eloquent are basically the same, so we take the cards has notes relationship to explain how it works.

好吧, 怎么使用`Eloquent`关联模型基本一致, 所以我们就使用 `cards` has `notes` 的关系来解释.

## `Get`/`Query`

- **`No Eloquent`**

```
    php artisan tinker
    # 1. Prepare one card whose id = 1
    $card = new App\Models\Card;
    $card->user_id = 1;    // assume it's 1 here
    $card->title   = '1st card';
    $card->save();
    
    # 2. Create a note with it's card_id = 1
    $note = new App\Models\Note;
    $note->card_id = 1;
    $note->content = '1st note';
    $note->save();
```

Yes, it just manually relate their relationship. It’s a little bit inelegant.


- **`With Eloquent`**
  - define the `notes()` method in Card Model
  
  ```
    public function notes()
    {
    	return $this->hasMany(Note::class);    // either `Note::class` or `\App\Models\Note`
    }
    public function path()
    {
      return '/cards/' . $this->id;
    }
  ```
  当我们使用 `\App\Models\Notes` 将转化成对象, 使用 `Notes::class`将转化成**类的字符串表示形式**。
  
  - call the `notes()` method in Card Model instance
  
  ```
    php artisan tinker
    
    $card = App\Models\Card::first()
    
    $card->notes->first()        # as collection
    
    # Or
    # $card->notes[0]        # as array
    
    # Or
    # $card->notes()->first()    # as object
  ```
  
  集合和对象的不同在于最后的查询`sql`
  
  通过一个测试证明下:
  
  ```
    php artisan tinker   # refresh the modifications in project
    
    # Setup a listener when query event occurs
    >>> DB::listen(function ($query){ echo $query->sql; }
    
    >>> $card = new App\Models\Card::first()
    select * from "cards" limit 1;
    
    # First execute the get-card-notes query
    >>> $card->notes
    select * from "notes" where "notes"."card_id" = ? and "note"."card_id" is not null
    
    # Re-execute the same query
    >>> $card->notes
    # You will see nothing here because the last query result is stored in the $card object and fetch them directly from object rather than the database
    
    # Check if the $card object has the notes relationships here
    >>> $card
    # You will see the notes of this card here
    
    # Check if notes relationship still exists after we refresh the object
    >>> $card->fresh()
    select * from "cards" where "id" = ? limit 1
    >>> $card
    # You wil note see the notes of this card here any more because the resuls cache are erased after `fresh()`
    
    # If we use as collection, the sql of get first note record might be like this:
    >>> $card->fresh()->notes->first()
    select * from "cards" where "id" = ? limit 1
    select * from "notes" where "notes"."card_id" = ? and "notes"."card_id" is not null
    
    # If we use as object method, the sql of get first note record might be like this:
    >>> $card->fresh()->notes()->first()
    select * from "cards" where "id" = ? limit 1
    select * from "notes" where "notes"."card_id" = ? and "notes"."card_id" is not null limit 1
  ```
  
  测试结果和上面的结论一致.

## `hasMany` and `belongsTo`

> Moreover, we can say that note also belongs to card. So here is how eloquent deal with this `belongs-to` relationship here.

**简单来说:就是 谁拥有谁==谁属于谁**

- - define the `card()` method in Note Model:
  
  ```
    public function card()
    {
      return $this->belongsTo(Card::class);
    }
  ```
  
- - call the `card` method in Note Model instance
  
  ```
    php artisan tinker
    
    $note = App\Models\Note::first()
    
    $note->card
  ```
> **Be aware that**: `hasMany()` related method is `notes()`, which is resonable because one card can has many notes, and `belongsTo()` related method is `card()`, which is resonable because one note only belongs to one card.

注意到: `hasMany()` 对应的方法是 `notes` (复数),是因为 **一张卡可以有很多笔记**, `belongTo` 对应的方法是 `card()` (单数),是因为 **一个笔记只属于一张卡**
    
你不应该创建方法名为 `note()` 或 `cards()`，因为它没有任何意义，必须回应你叫它什么方法。

**记住**，`hasmany()` 和复数的方法有关，`belongsto()`和单数的方法相关。

## `Create`/`save`

```
    php artisan tinker
    
    # Create a new Note
    $note = new App\Models\Note
    $nots->content = 'second note'
    
    # Decide which card to associate with
    $card = App\Models\Card::first()
    
    # Save a exists note by `save()`
    $card->notes()->save($note)    # we cann't use $card->notes->save() here
    
    # Or create a new note by `create()`
    $card->notes()->create(['content'=>'third note'])    # make sure that `content` is fillable in Note Model
    
    # See the new note associated with first card
    $card->notes
    # Yout will see new note was already in $card object
```

**Notice** that here we didn’t set the `card_id` for `$note`, because eloquent automatically did this for us.

**When we creating new record, we cann’t use collections any more, because it is only used for query-like operations.**

此外，当我们通过 `create()`直接创建一个新的记录, 需要某个字段批量赋值如:`content`, 需要在`$fillable`定义.

```
    class Note extends Model
    {
    	protected $fillable = ['content'];
    	
        public function card()
        {
        	return $this->belongsTo(Card::class);
        }
    }
```

`$fillable` 可以作为设置被批量赋值的属性的「白名单」,  这也是`Laravel`中避免攻击的方法之一.

# `Form`

基于之前的例子:我们有 `resources/views/cards/show.blade.php `

```
    @extends('layouts.main')
    @section('title', 'show card')
    @section('content')
    <div class="row">
    	<div class="col-md-6">
    		<h1>
    			{{ $card->title }}
    		</h1>
    		<ul class="list-group">
    			@foreach ($card->notes as $note)
    				<li class="list-group-item">
    					{{ $note->content }}
    				</li>
    			@endforeach
    		</ul>
    		<hr>
    		<h2>
    			Add More Notes Here
    		</h2>
    		<form method="POST" action="/cards/{{ $card->id }}/notes">
    		<input type="hidden" name="_token" value="{{ csrf_token() }}">
    			<div class="form-group">
    				<textarea name="content" class="form-control"></textarea>
    			</div>
    			<div class="form-group">
    				<button type="submit" class="btn btn-primary">
    					Add Note
    				</button>
    			</div>
    		</form>
    	</div>
    </div>
    @stop
```

创建 `routes.php`:

```
    Route::post('cards/{card}/notes', 'Notes@store');
```

创建控制器 `Notes@store`:

```
public function store(Request $request, Card $card)
    {
        // return $request->all();
        // return \Request::all();
        // return request()->all();
        // return $card;
        
  		# 1
        // $note = new Note;
        // $note->content = $request->content;
        // $card->notes()->save($note);
        
        # 2
        // $card->notes()->save([
        //     new Note([
        //         'content' => $request->content
        //         ])
        //     ]);
        
        # 3
        // $card->notes->create([
        //         'content' => $request->content
        //     ]);
        
        # 4
        $card->notes()->create(request()->all());      
  
        // return \Request::to('path/to/new/uri');
        // return redirect()->to('path/to/new/uri');
        return back();
    }
```

**注意:**
 
- - If you want use eloquent as an instance, the param name `$card` must **equals to** the route catch name `{card}`, here `card` equals `card`. Or `$card` in method will be nothing, which is unexpected.
  
- - When we use `create()` to insert an relationship, we must make sure that we had stipulated the `$fillable` in related model, as we talked before.

Since card and note has relationship here, so we can add a method to describle more clearer what we want to do, we can add a method `addNote()` in model Card, like this:

```
    public function addNote(Note $note)
    {
     	 $this->notes()->save($note);
    }
```

最后, 控制器中 `Notes@store()`:

```
    public function store(Request $request, Card $card)
    {
      	$card->addNote(
                    new Note($request->all())
                );

    	return back();
    }
```

## `RESTful Principles Review`

`RESTful`接口整个说明本表：

<u>[资源控制器](https://d.laravel-china.org/docs/5.5/controllers#resource-controllers)</u>

此外，由于浏览器只直接理解`GET`和`POST`方法，所以遵循`REST`约定可以像这样做:

```
    <form method="POST">
      {{ method_field('PATCH | DELETE | PUT') }}
      <!-- which equals to: -->
      <!-- <input type="hidden" name="method" value="PATCH"> -->
      <!-- more form elements -->
    </form>
```

## `Update` And `Eager Loading`

**预先加载**是避免大量的数据库查询该对象的数据可以得到一个查询通过建立`laravel`模型关系的有效途径。

> Before we using eager loading, if we want to get both the notes of one card, and the user this card belongs to, we may query *N+1* times like this (`notes` **table must has fileds named with `user_id` and `card_id`**):

```
    public function showByInstance(Card $card)
    {
    	return $card->notes[0]->user;
    }
```

**这里需要查询两次, 这是低效的**

使用预加载,我们可以将两次查询合并成一次,就像:

```
    public function showByInstance(Card $card)
    {
      return $card->load('notes.user');    // notes means Card@notes() user means Note@user()
      
      # Or
      // return $card->with('notes.user');
      
      # Or only load notes of this card
      // return $card->load('notes');
      // return $card->with('note')->find(1);    // 1 means this id of the card we want to find
    }
```

## `Validation`

- - `Form input`

```
    public function addNote(Request $request, Card $card)
    {
      $this->validate($request,[
        'content' => 'required|min:10'
      ]);     // Controller => ValidatesRequests@validate
    }
```

- - `CSRF`: See detial in FAQ.

- - `Middleware`

```
    Route::group([
    	'middleware' => ['web'],
    ], function () {
    	Route::post('cards/{card}/notes', 'Cards@addNote');
    	// Route::post('cards/{card}/notes', 'Notes@store');
      	// More routes
    });
```

- - `Errors`

每个视图都可以共享($errors)错误变量。

- - `old('field_name')`

> When form check fails, the preview wrong data are `withInput()` in the redirect, and we can put it into the right elements.

通过 `old()`,获取上一次输入的内容.

## FAQ

- <u>[php artisan migrate:reset failed to open stream: No such file or directory?](https://stackoverflow.com/questions/25237415/php-artisan-migratereset-failed-to-open-stream-no-such-file-or-directory)</u>

```
    composer dump-autoload
```

- <u>[TokenMismatchException in VerifyCsrfToken.php Line 67?](https://stackoverflow.com/questions/34866404/tokenmismatchexception-in-verifycsrftoken-php-line-67)</u>

```
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
    <!-- Or -->
    {{ csrf_field() }}
    <!-- Or set into meta tag -->
    <meta name="csrf-token" content="{{ csrf_token() }}">
```

- <u>[What’s the differences between PUT and PATCH?](https://laracasts.com/discuss/channels/general-discussion/whats-the-differences-between-put-and-patch?page=1)</u>

```
    PUT = replace the ENTIRE RESOURCE with the new representation provided (no mention of related resources in the spec from what i can see)
    // 用所提供的新表示替换整个资源（在我所看到的规范中没有提到相关资源）
    
    PATCH = replace parts of the source resource with the values provided AND|OR other parts of the resource are updated that you havent provided (timestamps) AND|OR updating the resource effects other resources (relationships)

```

## 重置和刷新命令列表

- `php artisan clear-compiled`: 删除已编译的类文件

- `php artisan auth:clear-resets `: 刷新过期密码重置令牌

- `php artisan cache:clear`: 刷新应用程序缓存

- `php artisan config:clear`: 删除配置缓存文件

- `php artisan migrate:refresh`: 重置并重新运行所有迁移

**Same with `migrate:rollback`, they will erase the data in database.**

- `php artisan migrate:reset`: 回滚所有数据库迁移

- `php artisan queue:flush`: 刷新所有失败的队列作业

- `php artisan queue:forget`: 删除失败的队列作业

- `php artisan route:clear`: 删除路由缓存文件

- `php artisan view clear`: 清除所有的编译文件


# 参考(References)

- [Laracasts](https://laracasts.com/series/laravel-5-from-scratch)
- [Laravel 5.6(英文)](https://laravel.com/docs/5.6)
- [Laravel 5.5(中文)](https://d.laravel-china.org/docs/5.5)
- [Laravel 维基百科](https://en.wikipedia.org/wiki/Laravel)
- [Laravel/Lumen](https://lumen.laravel-china.org/docs/5.3/installation)
- [Your One-Stop Guide to Laravel Commands](https://code.tutsplus.com/tutorials/your-one-stop-guide-to-laravel-commands--net-30349)

---
title: Eloquent:ORM of Laravel
date: 2018-02-10 19:01:20
categories: Laravel
tags: [Laravel, ELoquent, ORM]
---

> In simple, it’s just the `M` layer in the `MVC` design pattern.

<!--more-->

写给自己看的, 路人请转向 [文档](https://d.laravel-china.org/docs/5.5/eloquent), 目前内容比较多,后续慢慢用自己的话总结.

# 简介

> `Laravel` 的 `Eloquent ORM` 提供了漂亮、简洁的 `ActiveRecord` 实现来和**数据库交互**。每个数据库表都有一个对应的「模型」用来与该表交互。你可以通过模型查询数据表中的数据，并将新记录添加到数据表中。

在开始之前，请确保在 config/database.php 中配置数据库连接。

# 快速入门

## 定义模型

所有的 Eloquent 模型都继承了 `Illuminate\Database\Eloquent\Model` 类。

创建模型实例的最简单方法是使用 `Artisan` 命令 `make:model`：

```
    php artisan make:model User
```

如果要在生成模型时生成 数据库迁移，可以使用 `--migration` 或 `-m` 选项：

```
    php artisan make:model User --migration
    
    php artisan make:model User -m
```

## 简单例子

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * 与模型关联的数据表
         *
         * @var string
         */
        protected $table = 'my_flights';
        
        protected $primaryKey = 'my_id';
        
        /**
         * 该模型是否被自动维护时间戳
         *
         * @var bool
         */
        public $timestamps = false;
        
        /**
         * 模型的日期字段的存储格式
         *
         * @var string
         */
        protected $dateFormat = 'U';
        
        // 自定义用于存储时间戳的字段名
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
        
        /**
         * 此模型的连接名称。
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }
```

## 检索多个模型

创建完模型 及其关联的数据表 之后，就可以开始从数据库中检索数据。可把每个 `Eloquent` 模型想像成强大的 查询构造器，它让你可以流畅地查询与该模型相关联的数据库表

```
    <?php
    
    use App\Flight;
    
    $flights = App\Flight::all();
    
    foreach ($flights as $flight) {
        echo $flight->name;
    }
```

### 添加其他约束

```
    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();
```

### 集合

使用 Eloquent 中的方法比如 `all` 和 `get` 可以检索多个结果，并且会返回一个 `Illuminate\Database\Eloquent\Collection` 实例

`Collection` 类提供了 [很多辅助函数](https://d.laravel-china.org/docs/5.5/eloquent-collections#available-methods) 来处理Eloquent 结果。

```
    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });
```

你也可以像数组一样简单地来遍历集合：

```
    foreach ($flights as $flight) {
        echo $flight->name;
    }
```

### 分块结果

`chunk` 方法会检索 `Eloquent` 模型的「分块」，将它们提供给指定的 `Closure` 进行处理。在处理大型结果集时，使用 `chunk` 方法可节省内存：
    
```
    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });
```

传递到方法的第一个参数是希望每个「分块」接收的**数据量**。**闭包**则被作为第二个参数传递，它会在每次执行数据库查询传递每个块时被调用。

### 使用游标

`cursor` 允许你使用游标来遍历数据库数据，该游标只执行一个查询。处理大量数据时，可以使用 `cursor` 方法可以大幅度减少内存的使用量：

```
    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }
```

## 检索单个模型／集合

除了从指定的数据表检索所有记录外，你也可以通过 `find` 或 `first` 方法来检索单条记录。这些方法不是返回一组模型，而是返回一个模型实例：

```
    // 通过主键取回一个模型...
    $flight = App\Flight::find(1);
    
    // 取回符合查询限制的第一个模型 ...
    $flight = App\Flight::where('active', 1)->first();
```

你也可以用主键数组为参数调用 find 方法，它将返回匹配记录的集合：

```
    $flights = App\Flight::find([1, 2, 3]);
```

### 「找不到」异常

如果你希望在找不到模型时抛出异常，可以使用 `findOrFail` 以及 `firstOrFai`l 方法。这些方法会检索查询的第一个结果。如果没有找到相应结果，就会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`：

```
    $model = App\Flight::findOrFail(1);
    
    $model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

如果没有对异常进行捕获，则会自动返回 `HTTP 404` 响应给用户。也就是说，在使用这些方法时，不需要另外写个检查来返回 `404` 响应：

```
   Route::get('/api/flights/{id}', function ($id) {
       return App\Flight::findOrFail($id);
   });
```

### 检索集合

使用 [聚合函数](https://d.laravel-china.org/docs/5.5/queries#aggregates)

```
    $count = App\Flight::where('active', 1)->count();
    
    $max = App\Flight::where('active', 1)->max('price');
```

## 插入 & 更新模型

### 插入

```
    <?php
    
    namespace App\Http\Controllers;
    
    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class FlightController extends Controller
    {
        /**
         * 创建一个新的航班实例。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // 验证请求...
    
            $flight = new Flight;
    
            $flight->name = $request->name;
    
            $flight->save();
        }
    }
```

### 更新

```
    $flight = App\Flight::find(1);
    
    $flight->name = 'New Flight Name';
    
    $flight->save();
```

#### 批量更新

```
    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);
```

### 批量赋值

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * 可以被批量赋值的属性。
         *
         * @var array
         */
        protected $fillable = ['name'];
    }
```

只有我们设置好可以被批量赋值的属性，才能通过 `create` 方法来为数据库添加新记录到。create 方法会返回已保存的模型实例：

```
    $flight = App\Flight::create(['name' => 'Flight 10']);
```

如果你已经有一个模型实例，你可以传递数组给 `fill` 方法：

```
    $flight->fill(['name' => 'Flight 22']);
```

#### **保护属性**

`$fillable` 可以作为设置被批量赋值的属性的「白名单」，同样的 `$guarded` 属性也可以实现这个需求。但不同的是，`$guarded` 属性包含的是不想被批量赋值的属性的数组

使用的时候，也要注意只能是 `$fillable` 或 `$guarded` 二选一

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * 不可被批量赋值的属性。
         *
         * @var array
         */
        protected $guarded = ['price'];
    }
```

如果想让所有的属性都可以被批量赋值，就把 `$guarded` 定义为空数组。

```
    /**
     * 不可被批量赋值的属性。
     *
     * @var array
     */
    protected $guarded = [];
```

## 其他创建方法

### `firstOrCreate` / `firstOrNew`

- `firstOrCreate` 方法会使用给定的字段及其值在数据库中查找记录。如果在数据库中找不到模型，则将使用第一个参数中的属性以及可选的第二个参数中的属性插入记录。
- `firstOrNew` 方法就类似 `firstOrCreate` 方法，会在数据库中查找匹配给定属性的记录。如果模型未被找到，则会返回一个新的模型实例。请注意，在这里面，`firstOrnew` 返回的模型还尚未保存到数据库，必须要手动调用 `save` 方法才能保存它：

```
    // 通过 name 属性检索航班，当结果不存在时创建它...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
    
    // 通过 name 属性检索航班，当结果不存在的时候用 name 属性和 delayed 属性去创建它
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );
    
    // 通过 name 属性检索航班，当结果不存在时实例化...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
    
    // 通过 name 属性检索航班，当结果不存在的时候用 name 属性和 delayed 属性实例化
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );    
```

### `updateOrCreate`

```
    // 如果不存在匹配的模型就创建一个
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    )
```

## 删除模型

```
    $flight = App\Flight::find(1);
    
    $flight->delete();
```

### 通过主键删除模型

```
    App\Flight::destroy(1);
    
    App\Flight::destroy([1, 2, 3]);
    
    App\Flight::destroy(1, 2, 3);
```

### 通过查询删除模型

```
    $deletedRows = App\Flight::where('active', 0)->delete();
```

### 软删除

要启动模型上的软删除，则必须在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` trait 并添加 `deleted_at` 字段到 `$dates` 属性上：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;
    
    class Flight extends Model
    {
        use SoftDeletes;
    
        /**
         * 需要被转换成日期的属性。
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }
```

你也应该添加 `deleted_at` 字段到数据表中。Laravel [结构生成器](https://d.laravel-china.org/docs/5.5/migrations) 包含了一个辅助函数用来创建此字段：

```
    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });
```

要给定模型实例是否已被软删除，可以使用 `trashed` 方法：

```
    if ($flight->trashed()) {
        //
    }
```

### 查询被软删除的模型

#### 包括被软删除的模型

```
    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();
```

`withTrashed` 方法也可用于 [关联](https://d.laravel-china.org/docs/5.5/eloquent-relationships) 查询：

#### 只检索被软删除的模型

```
    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();
```

#### 恢复被软删除的模型

如果想「取消删除」被软删除的模型。可在模型实例上使用 `restore` 方法将一个被软删除的模型恢复到有效状态：

```
    $flight->restore();
```

你也可以在查询上使用 `restore` 方法来快速地恢复多个模型。像其他「批量赋值」操作一样，这并不会触发任何模型事件：

```
    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();
```

与 `withTrashed` 方法类似，`restore` 方法也可以被用在 关联 查询上:

```
    $flight->history()->restore();
```

#### 永久删除模型

如果要真正地从数据库中永久删除软删除的模型，可以使用 `forceDelete` 方法：

```
    // 强制删除单个模型实例...
    $flight->forceDelete();
    
    // 强制删除所有相关模型...
    $flight->history()->forceDelete();
```


# 模型关联

## 定义关联

`Eloquent` 关联在 `Eloquent` 模型类中以方法的形式呈现。如同 `Eloquent` 模型本身，关联也可以作为强大的 [查询语句构造器](https://d.laravel-china.org/docs/5.5/queries) 使用，提供了强大的链式调用和查询功能。

### 一对一

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * 获得与用户关联的电话记录。
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }
```

`hasOne` 方法的第一个参数是关联模型的类名, 可以像在访问模型中定义的属性一样，使用动态属性：

```
    $phone = User::find(1)->phone;
```

可以通过给 `hasOne` 方法传递第二个参数覆盖默认使用的外键名：

```
    return $this->hasOne('App\Phone', 'foreign_key');
```

此外，Eloquent 假定外键值是与父级 `id`（或自定义 ``）列的值相匹配的。 换句话说，Eloquent 将在 `Phone` 记录的 `user_id` 列中查找与用户表的 id 列相匹配的值。 如果您希望该关联使用 `id` 以外的自定义键名，则可以给 `hasOne` 方法传递第三个参数：

```
    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```

#### 定义反向关联

`hasOne` 方法对应的 `belongsTo` 方法：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Phone extends Model
    {
        /**
         * 获得拥有此电话的用户。
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }
```

在上面的例子中，Eloquent 会尝试匹配 `Phone` 模型上的 `user_id` 至 `User` 模型上的 `id`。 它是通过检查关系方法的名称并使用 `_id` 作为后缀名来确定默认外键名称的。 但是，如果 `Phone` 模型的外键不是`user_id`，那么可以将自定义键名作为第二个参数传递给 `belongsTo` 方法：

```
    /**
     * 获得拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }
```

如果父级模型没有使用 `id` 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 `belongsTo` 方法传递第三个参数的形式指定父级数据表的自定义键：

```
    /**
     * 获得拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }
```

#### 默认模型

```
    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }
```

您也可以通过传递数组或者使用闭包的形式，填充默认模型的属性：

```
    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => '游客',
        ]);
    }
    
    /**
     * 获得此文章的作者。
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = '游客';
        });
    }
```

### 一对多

「一对多」关联用于定义单个模型拥有任意数量的其它关联模型。例如，一篇博客文章可能会有无限多条评论

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        /**
         * 获得此博客文章的评论。
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }
```

联关系定义好后，我们就可以通过访问 `comments` 属性获得评论集合。记住，因为 Eloquent 提供了「动态属性」，所以我们可以像在访问模型中定义的属性一样，访问关联方法：

```
    $comments = App\Post::find(1)->comments;
    
    foreach ($comments as $comment) {
        //
    }
```

形如 `hasOne` 方法，您也可以在使用 `hasMany` 方法的时候，通过传递额外参数来覆盖默认使用的外键与本地键。

```
    return $this->hasMany('App\Comment', 'foreign_key');
    
    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
```

### 一对多（反向）

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Comment extends Model
    {
        /**
         * 获得此评论所属的文章。
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }
```

关联关系定义好后，我们就可以在 `Comment` 模型上使用 `post` 「动态属性」获得 `Post` 模型了。

```
    $comment = App\Comment::find(1);
    
    echo $comment->post->title;
```

如果 Comment 模型的外键不是 post_id，那么可以将自定义键名作为第二个参数传递给belongsTo方法：

```
    /**
     * 获得此评论所属的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }
```

如果父级模型没有使用 id 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 `belongsTo`方法传递第三个参数的形式指定父级数据表的自定义键：

```
    如果父级模型没有使用 id 作为主键，或者是希望用不同的字段来连接子级模型，则可以通过给 belongsTo方法传递第三个参数的形式指定父级数据表的自定义键：
```

### 多对多

```
    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
```

- 为了确定连接表表名，Eloquent 会按照字母顺序合并两个关联模型的名称。 当然，您可以自由地覆盖这个约定，通过给 belongsToMany 方法指定第二个参数实现：

- 第三个参数是定义此关联的模型在连接表里的键名

- 第四个参数是另一个模型在连接表里的键名：

### 定义反向关联

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Role extends Model
    {
        /**
         * 获得此角色下的用户。
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }
```



## 查询关联

### 关联方法 Vs. 动态属性

```
    $user = App\User::find(1);
    
    foreach ($user->posts as $post) {
        //
    }
```

动态属性是「懒加载」的，意味着它们的关联数据只在实际被访问时才被加载

### 基于存在的关联查询

```
    // 获得所有至少有一条评论的文章...
    $posts = App\Post::has('comments')->get();
    
    // 获得所有有三条或三条以上评论的文章...
    $posts = Post::has('comments', '>=', 3)->get();
    
    // 获得所有至少有一条获赞评论的文章...
    $posts = Post::has('comments.votes')->get();
    
    // 获得所有至少有一条评论内容满足 foo% 条件的文章
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();
```

### 基于不存在的关联查询

```
    $posts = App\Post::doesntHave('comments')->get();
    
    $posts = Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();
```

### 关联数据计数

```
    // 只想统计结果数而不需要加载实际数据
    $posts = App\Post::withCount('comments')->get();
    
    foreach ($posts as $post) {
        echo $post->comments_count;
    }
    
    // 可以为多个关联数据「计数」，并为其查询添加约束条件：
    $posts = Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();
    
    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;
    
    // 可以为关联数据计数结果起别名，允许在同一个关联上多次计数：
    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();
    
    echo $posts[0]->comments_count;
    
    echo $posts[0]->pending_comments_count;
```

## 预加载

当作为属性访问 Eloquent 关联时，关联数据是「懒加载」的。意味着在你第一次访问该属性时，才会加载关联数据。不过，是当你查询父模型时，Eloquent 可以「预加载」关联数据。预加载避免了 N + 1 查询问题。要说明 N + 1 查询问题

## 插入 & 更新关联模型

# Eloquent Collection

`Eloquent` 返回的所有多结果集都是 `Illuminate\Database\Eloquent\Collection` 对象的实例，

> 大多数 `Eloquent` 集合方法会返回新的 `Eloquent` 集合实例，但是 `pluck`, `keys`, `zip`, `collapse`, `flatten` 和 `flip` 方法除外，它们会返回 集合基类 实例。同样，如果 `map` 操作返回的集合不包含任何 `Eloquent` 模型，那么它会被自动转换成集合基类。

## 可用的方法

> https://d.laravel-china.org/docs/5.5/eloquent-collections#available-methods

# 修改器

## 简介
当你在 `Eloquent` 模型实例中获取或设置某些属性值的时候，访问器和修改器允许你对 `Eloquent` 属性值进行格式化。

## 访问器 & 修改器

### 定义一个访问器

若要定义一个访问器，则须在你的模型上创建一个 `getFooAttribute` 方法。要访问的 `Foo` 字段需使用**「驼峰式」**来命名。

在这个例子中，我们将为 `first_name` 属性定义一个访问器。当 `Eloquent` 尝试获取 `first_name` 的值时，将会自动调用此访问器：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * 获取用户的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }
```

```
    $user = App\User::find(1);
    
    $firstName = $user->first_name;
```

当然，你也可以通过已有的属性，使用访问器返回新的计算值：

```
    /**
     * 获取用户名全称
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
      return "{$this->first_name} {$this->last_name}";
    }
```

### 定义一个修改器

若要定义一个修改器，则须在模型上定义一个 `setFooAttribute` 方法。要访问的 `Foo` 字段需使用「驼峰式」来命名.

让我们再来定义 `first_name` 属性的修改器。当我们尝试在模型上设置 `first_name` 的值时，该修改器将被自动调用：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * 设定用户的名字。
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }
```

修改器会获取属性已经被设置的值，允许你操作该值并将其设置到 `Eloquent` 模型内部的 `$attributes` 属性上。

举个例子，如果我们尝试将 `first_name` 属性设置成 `Sally`：

```
    $user = App\User::find(1);
    
    $user->first_name = 'Sally';
```

## 日期转换器

默认情况下，`Eloquent` 将会把 `created_at` 和 `updated_at` 字段转换成 [Carbon](https://github.com/briannesbitt/Carbon) 实例，它继承了 PHP 原生的 DateTime 类，并提供了各种有用的方法。

你可以通过重写模型的 $dates 属性，自行定义哪些日期类型字段会被自动转换，或者完全禁止所有日期类型字段的转换：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * 应被转换为日期的属性。
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }
```

当某个字段被认为是日期格式时，你或许想将其数值设置成一个 `UNIX` 时间戳、日期字符串（`Y-m-d`）、日期时间（ `date-time` ）字符串，当然还有 `DateTime` 或 `Carbon` 实例，并且让日期值自动正确地保存到你的数据库中：

```
    $user = App\User::find(1);
    
    $user->deleted_at = Carbon::now();
    
    $user->save();
```

就如上面所说的，当获取到的属性包含在 `$dates` 属性时，都将会自动转换成 `Carbon` 实例，允许你在属性上使用任意的 `Carbon` 方法：

```
    $user = App\User::find(1);
    
    return $user->deleted_at->getTimestamp();
```

# API 资源

## 生成资源

你可以使用 `make:resource Artisan` 命令来生成资源类。默认情况下生成的资源都会被放置在框架 `app/Http/Resources` 文件夹下。资源继承 `Illuminate\Http\Resources\Json\Resource` 类：

```
    php artisan make:resource User
```

### 资源集合

```
    php artisan make:resource Users --collection
    
    php artisan make:resource UserCollection
```

# 序列化

## 序列化模型 & 集合

### 序列化成数组

```
    $user = App\User::with('roles')->first();
    
    return $user->toArray();
```

### 序列化成 `JSON`

```
    $user = App\User::find(1);
    
    return $user->toJson()
```

## 隐藏来自 `JSON` 的属性

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }
```

另外，你也可以使用 `visible` 属性来定义应该包含在你的模型数组和 `JSON` 表示中的属性白名单。白名单外的其他属性将隐藏，不会出现在转换后的数组或 `JSON` 中：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }
```

### 临时修改属性的可见度

```
    return $user->makeVisible('attribute')->toArray();
```

相应的，你可以在模型实例后使用 makeHidden 方法来隐藏通常显示的属性：

```
    return $user->makeHidden('attribute')->toArray();
```

## 添加参数到 `JSON` 中

首先你需要为这个值定义一个 访问器：

```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }
```

访问器创建成功后，只需添加该属性到该模型的 `appends` 属性中。注意，属性名称通常遵循 `「Snake Case」`, 的命名方式，即是访问器的名称是基于 `「Camel Case」` 的命名方式。


```
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }
```

---
title: 数据库请求构建器
date: 2018-02-11 16:30:04
categories: Framework
tags: [Laravel, Database, Query]
---

Laravel 的数据库查询构造器提供了一个方便的接口来创建及运行数据库查询语句。它能用来执行应用程序中的大部分数据库操作，且能在所有被支持的数据库系统中使用。

Laravel 的查询构造器使用 PDO 参数绑定来保护你的应用程序免受 SQL 注入的攻击。因此没有必要清理作为绑定传递的字符串。

<!--more-->

# 获取结果

## 从数据表中获取所有的数据

```
    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * 显示所有应用程序用户的列表
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();
    
            return view('user.index', ['users' => $users]);
        }
    }
```

`get` 方法会返回一个包含 `Illuminate\Support\Collection` 的结果，其中每个结果都是一个 `PHP StdClass` 对象的一个实例。你可以通过访问字段作为对象的属性来访问每列的值：

```
    get 方法会返回一个包含 Illuminate\Support\Collection 的结果，其中每个结果都是一个 PHP StdClass 对象的一个实例。你可以通过访问字段作为对象的属性来访问每列的值：
```

## 从数据表中获取单个列或行

```
    $user = DB::table('users')->where('name', 'John')->first();
    
    echo $user->name;
```

如果你甚至不需要整行数据，就使用 value 方法从记录中取出单个值。该方法将直接返回字段的值：

```
    $email = DB::table('users')->where('name', 'John')->value('email');
```

## 获取一列的值

```
    $titles = DB::table('roles')->pluck('title');
    
    foreach ($titles as $title) {
        echo $title;
    }
```

你也可以在返回的集合中指定字段的自定义键值：

```
    $roles = DB::table('roles')->pluck('title', 'name');
    
    foreach ($roles as $name => $title) {
        echo $title;
    }
```

## 结果分块

如果你需要操作数千条数据库记录，可以考虑使用 chunk 方法。这个方法每次只取出一小块结果传递给 闭包 处理，这对于编写数千条记录的 Artisan 命令 而言是非常有用的

```
    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });
```

你可以从 `闭包` 中返回 `false` 来阻止进一步的分块的处理：

```
    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...
    
        return false;
    });
```

## 聚合

```
    $users = DB::table('users')->count();
    
    $price = DB::table('orders')->max('price');
```

当然，你也可以将这些方法和其它语句结合起来：

```
    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');
```

# `Selects`

## 指定一个 `Select` 语句

```
    $users = DB::table('users')->select('name', 'email as user_email')->get();
```

`distinct` 方法允许你强制让查询返回不重复的结果：

```
    $users = DB::table('users')->distinct()->get();
```

如果你已有一个查询构造器实例，并且希望在现有的 `select` 语句中加入一个字段，则可以使用 `addSelect` 方法：

```
    $query = DB::table('users')->select('name');
    
    $users = $query->addSelect('age')->get();
```

# 原生表达式

使用 `DB::raw` 方法可以创建原生表达式

```
    $users = DB::table('users')
               ->select(DB::raw('count(*) as user_count, status'))
               ->where('status', '<>', 1)
               ->groupBy('status')
               ->get();
```

**原生表达式将会被当作字符串注入到查询中，所以要小心避免创建 SQL 注入漏洞。**

# 原生方法

可以使用以下的方法代替 `DB::raw` 将原生表达式插入查询的各个部分。

## `selectRaw`

```
    // 第二个参数接受一个可选的绑定参数的数组：
    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();
```

## `whereRaw` / `orWhereRaw`

```
    // 这些方法接受一个可选的绑定数组作为他们的第二个参数：
    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();
```

## `havingRaw` / `orHavingRaw`

```
    // 可用于将原生字符串设置为 having 语句的值
    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();
```

## `orderByRaw`

```
    // 可用于将原生字符串设置为 order by 语句的值
    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();
```


# `Joins`

## `Inner Join` 语句

```
    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();
```

## `Left Join` 语句

```
    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();
```

## `Cross Join` 语句

使用 crossJoin 方法和你想要交叉连接的表名来做「交叉连接」。交叉连接在第一个表和连接之间生成笛卡尔积：

```
    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();
```

## 高级 `Join` 语句

你也可以指定更高级的 `join` 语句。比如传递一个 闭包 作为 `join` 方法的第二个参数。此 闭包 接收一个 `JoinClause` 对象，从而在其中指定 `join` 语句中指定约束：

```
    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();
```

如果你想要在连接上使用`「where」`风格的语句，可以在连接上使用 `where` 和 `orWhere` 方法。这些方法可以用来比较值和对应的字段：

```
    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();
```

# `Unions`

查询构造器还提供了将两个查询「合并」起来的快捷方式

```
    $first = DB::table('users')
                ->whereNull('first_name');
    
    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();
```

# `Where` 语句

## 简单的 `Where` 语句

```
    $users = DB::table('users')->where('votes', '=', 100)->get();
    
    # Or
    $users = DB::table('users')->where('votes', 100)->get();
    
    # Or
    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();
    
    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();
    
    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();
                    
     # Or
     $users = DB::table('users')->where([
         ['status', '=', '1'],
         ['subscribed', '<>', '1'],
     ])->get();
```

## `Or` 语句

```
    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();
```

## 其它 `Where` 语句

### `whereBetween`

```
    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();
```

### `whereNotBetween`

```
    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();
```

### `whereIn` / `whereNotIn`

```
    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();
                        
    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();
```

### `whereNull` / `whereNotNull`

```
    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();
                        
    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();
```

### `whereDate` / `whereMonth` / `whereDay` / `whereYear` / `whereTime`

```
    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();
    
    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();
                    
    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();
                    
    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();
                    
    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20')
                    ->get();
```

### `whereColumn`

`whereColumn` 方法用于验证两个字段是否相等：

```
    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();
```

还可以将比较运算符传递给该方法：

```
    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();
```

`whereColumn` 方法也可以传递一个包含多个条件的数组。这些条件将使用 `and` 运算符进行连接：

```
    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();
```

### 参数分组

```
    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();
```

# `Where Exists` 语句

`whereExists` 方法允许你编写 `where exists` SQL 语句

```
    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();
```

# `JSON where` 语句

Laravel 也支持查询 JSON 类型的字段（仅在对 JSON 类型支持的数据库上）。目前，本特性仅支持 MySQL 5.7+ 和 Postgres数据库。可以使用 `->` 运算符来查询 JSON 列数据：

```
    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();
    
    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();
```

# `Ordering`, `Grouping`, `Limit`, & `Offset`

## orderBy

```
    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();
```

## `latest` / `oldest`

`latest` 和 `oldest` 方法允许你轻松地按日期对查询结果排序。默认情况下是对 `created_at` 字段进行排序。或者，你可以传递你想要排序的字段名称：

```
    $user = DB::table('users')
                    ->latest()
                    ->first();
```

## `inRandomOrder`

`inRandomOrder` 方法可以将查询结果随机排序

```
    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();
```

## `groupBy` / `having`

```
    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();
```

可以将多个参数传递给 groupBy 方法，按多个字段进行分组：

```
    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();
```

## `skip` / `take`

```
    $users = DB::table('users')->skip(10)->take(5)->get();
```

或者，你也可以使用 limit 和 offset 方法：

```
    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();
```

# 条件语句

有时你可能想要子句只适用于某个情况为真时才执行查询

```
    $role = $request->input('role');
    
    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();
```

只有当 `when` 方法的第一个参数为 `true` 时，闭包里的 `where` 语句才会执行。如果第一个参数是 `false`，这个闭包将不会被执行。


你可以将另一个闭包当作第三个参数传递给 `when` 方法。如果第一个参数的值为 `false` 时，这个闭包将执行。为了说明如何使用此功能，我们将使用它配置查询的默认排序：

```
    $sortBy = null;
    
    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();
```

# `Inserts`

查询构造器也提供了将记录插入数据库表的 insert 方法

```
    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );
```

你还可以在 `insert` 中传入一个嵌套数组向表中插入多条记录。每个数组代表要插入表中的行：

```
    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);
```

## 自增 ID

若数据表存在自增的 ID，则可以使用 `insertGetId` 方法来插入记录然后获取其 ID：

```
    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );
```

# `Updates`

```
    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);
```

## 更新 `JSON` 字段

```
    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);
```

## 自增 & 自减

```
    DB::table('users')->increment('votes');
    
    DB::table('users')->increment('votes', 5);
    
    DB::table('users')->decrement('votes');
    
    DB::table('users')->decrement('votes', 5);
```

你也可以在操作过程中指定要更新的字段

```
    DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

# `Deletes`

查询构造器也可使用 `delete` 方法从数据表中删除记录

```
    DB::table('users')->delete();
    
    DB::table('users')->where('votes', '>', 100)->delete();
```

如果你需要清空表，你可以使用 `truncate` 方法，这将删除所有行，并重置自动递增 ID 为零：

```
    DB::table('users')->truncate();
```

# 悲观锁

查询构造器也包含一些可以帮助你在 `select` 语句上实现「悲观锁定」的函数 。若要在查询中使用 **「共享锁」**，可以使用 `sharedLock` 方法。

**共享锁可以防止选中的行被篡改，直到事务被提交为止：**

```
    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```

或者，你也可以使用 `lockForUpdate` 方法。使用**「更新」锁** 可避免行被其它共享锁修改或选取：

```
    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```
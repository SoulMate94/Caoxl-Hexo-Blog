---
title: 基于Laravel, 使用Redis做购物车
date: 2018-07-16 17:19:56
categories: Laravel
tags: [Laravel, Redis, 购物车, Framework]
---

> 三人行必有我师, 本文出自我的好朋友 [葉蕓榕: laravel与Redis开发购物车](http://yeyunrong.com/2018/03/29/laravel%E4%B8%8ERedis%E5%BC%80%E5%8F%91%E8%B4%AD%E7%89%A9%E8%BD%A6/) 的博客

<!-- more -->

> 我在这里就不做过多赘述, 只写一些必要的东西

# 环境准备

- 使用`laravel`内置的用户登陆认证系统并再其基础上进行修改

```
    php artisan make:auth
    
    php artisan migrate
```

- 安装`Redis`

```
    composer require predis/predis
```

# 直接上源码

## 路由准备

- `/routes/web.php`

```php
    # ShopCar Routes
    
    Route::get('shopping/add/{id}', 'ShoppingController@add')->name('shopping.add');
    
    Route::get('shopping/shop_car_info', 'ShoppingController@getShopCarInfo')->name('shopping.shop_car_info');
    
    Route::get('shopping/shop_add/{id}', 'ShoppingController@shopCarAdd')->name('shopping.shop_add');
    
    Route::get('shopping/shop_del/{id}', 'ShoppingController@shopCarDel')->name('shopping.shop_del');
    
    Route::get('shopping/shop_clean/{id}', 'ShoppingController@cleanShopCar')->name('shopping.shop_clean');
```


## 前端页面

- `home.blade.php`

```html
    @extends('layouts.app')
    
    @section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
    
                    {{--@include('shared.message')--}}
    
                    <div class="card-header">Dashboard</div>
    
                    <div class="card-body">
                        @if (session('status'))
                            <div class="alert alert-success" role="alert">
                                {{ session('status') }}
                            </div>
                        @endif
    
                        You are logged in!
                    </div>
                    <div class="card-body">
                        商品0001<a href="/shopping/add/1">加入购物车</a> <br>
                        商品0002<a href="/shopping/add/2">加入购物车</a> <br/>
                        商品0003<a href="/shopping/add/3">加入购物车</a>
                    </div>
    
                    <div class="card-body">
                        <a href="/shopping/shop_car_info">购物车</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
    @endsection
```

![home.blade.php](https://caoxl.com/images/shop_car_home.png)

- `shopcar.blade.php`

```html
    @extends('layouts.app')
    
    @section('content')
        <div class="container">
            <div class="row">
                <div class="col-md-8 col-md-offset-2">
                    <div class="card">
                        {{--@include('shared.messages')--}}
    
                        <div class="card-body">
                            @if($allGoodsKey)
                                @for($i = 0; $i < count($allGoodsKey); $i++)
                                    <p>
                                        商品ID: {{ $allGoodsInfo[$allGoodsKey[$i]]['id'] }} <br>
                                        商品昵称: {{ $allGoodsInfo[$allGoodsKey[$i]]['gname'] }} <br>
                                        商品价格: {{ $allGoodsInfo[$allGoodsKey[$i]]['price'] }} <br>
                                        商品数量: {{ $allGoodsNum[$allGoodsKey[$i]] }} <br>
                                        商品总价: {{ $allGoodsNum[$allGoodsKey[$i]] * $allGoodsInfo[$allGoodsKey[$i]]['price'] }}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    
                                        <a href={{ route('shopping.shop_add', $allGoodsInfo[$allGoodsKey[$i]]['id']) }}>+增加</a>&nbsp;&nbsp;
                                        <a href={{ route('shopping.shop_del', $allGoodsInfo[$allGoodsKey[$i]]['id']) }}>-减少</a>&nbsp;&nbsp;
                                        <a href={{ route('shopping.shop_clean', $allGoodsInfo[$allGoodsKey[$i]]['id']) }}>清除</a>&nbsp;&nbsp;
                                    </p>
                                @endfor
    
                            @else
                                <h2>购物车空空如也~</h2>
                            @endif
                        </div>
                        <div class="card-body">
                            <a href={{ route('home') }}>去购物</a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    @endsection
```

![shopcar.blade.php](https://caoxl.com/images/shop_car.png)

## 后端代码

- `ShoppingController`

```php
    <?php
    
    namespace App\Http\Controllers;
    
    use Illuminate\Support\Facades\Redis;
    use Illuminate\Support\Facades\Auth;
    
    class ShoppingController extends Controller
    {
        /**
         * 添加商品
         * @param $goods_id
         * @return \Illuminate\Http\RedirectResponse
         */
        public function add($goods_id)
        {
            $user = Auth::user();
    
            $users_table = Redis::exists('ShoppingCar :' . $user->id);
    
            if ($users_table) {
                // 用户表存在
                // 检查该商品ID是否存在
                $GoodsId = Redis::hexists('ShoppingCar :' . $user->id, $goods_id);
    
                if ($GoodsId) {
                    // 存在取出该商品数量, 进行+1操作
                    $num = Redis::hget('ShoppingCar :' . $user->id, $goods_id);
                    $newNum = $num + 1;
    
                    // 修改完之后的保存
                    Redis::hset('ShoppingCar :' . $user->id, $goods_id, $newNum);
                } else {
                    // 将数量设置为1
                    Redis::hset('ShoppingCar :' . $user->id, $goods_id, 1);
                }
            } else {
                // 用户表不存在
                Redis::hmset('ShoppingCar :' . $user->id, array($goods_id => 1));
            }
    
            $goods_info_table = Redis::hexists('GoodsInfo:Goods', $goods_id);
    
            if (!$goods_info_table) {
                // 先将商品数组信息取出并转换成Json格式
                Redis::hset('GoodsInfo:Goods', $goods_id, json_encode($this->getGoodsInfo($goods_id)));
            }
    
            // 通知用户
            session()->flash('Success', '添加成功');
    
            return redirect()->route('home');
        }
    
        /**
         * 获取用户全部商品
         * @return \Illuminate\Contracts\View\Factory|\Illuminate\View\View
         */
        public function getShopCarInfo()
        {
            $user = Auth::user();
    
            $allGoodsKey = Redis::hkeys('ShoppingCar :' . $user->id);
    
            if (null == $allGoodsKey) {
                $allGoodsNum  = false;
                $allGoodsInfo = false;
            }
    
            for ($i = 0; $i < count($allGoodsKey); $i++) {
                $allGoodsNum[$allGoodsKey[$i]]  = Redis::hget('ShoppingCar :' . $user->id, $allGoodsKey[$i]);
                $allGoodsInfo[$allGoodsKey[$i]] = json_decode(Redis::hget('GoodsInfo:Goods', $allGoodsKey[$i]), true);
            }
    
            return view('shopping_car.shopcar', [
                'allGoodsInfo' => $allGoodsInfo,
                'allGoodsNum'  => $allGoodsNum,
                'allGoodsKey'  => $allGoodsKey
            ]);
        }
    
        public function getGoodsInfo($goods_id)
        {
            $GoodsInfo= array(
    
                1 => array(
                    'id'    => 1,
                    'gname' => 'goods1',
                    'price' => '1'
                ),
    
                2 => array(
                    'id'    => 2,
                    'gname' => 'goods2',
                    'price' => '2'
                ),
                3 => array(
                    'id'    => 3,
                    'gname' => 'goods3',
                    'price' => '3'
                ),
                4 => array(
                    'id'    => 4,
                    'gname' => 'goods4',
                    'price' => '4'
                ),
            );
    
            return $GoodsInfo[$goods_id];
        }
    
        /**
         * 购物车内添加商品
         * @param $goods_id
         * @return \Illuminate\Contracts\View\Factory|\Illuminate\View\View
         */
        public function shopCarAdd($goods_id)
        {
            $user = Auth::user();
    
            if (!$user) {
                redirect()->route('home');
            }
    
            // 取出该商品数
            $num    = Redis::hget('ShoppingCar :' . $user->id, $goods_id);
            $newNum = $num + 1;
    
            // 修改完之后的保存
            Redis::hset('ShoppingCar :' . $user->id, $goods_id, $newNum);
    
            $allGoodsKey = Redis::hkeys('ShoppingCar :' . $user->id);
    
            for ($i = 0; $i < count($allGoodsKey); $i++) {
                $allGoodsNum[$allGoodsKey[$i]]  = Redis::hget('ShoppingCar :' . $user->id, $allGoodsKey[$i]);
                $allGoodsInfo[$allGoodsKey[$i]] = json_decode(Redis::hget('GoodsInfo:Goods', $allGoodsKey[$i]), true);
            }
    
            return view('shopping_car.shopcar', [
                'allGoodsInfo' => $allGoodsInfo,
                'allGoodsNum'  => $allGoodsNum,
                'allGoodsKey'  => $allGoodsKey
            ]);
        }
    
        /**
         * 购物车内减少商品
         * @param $goods_id
         * @return \Illuminate\Contracts\View\Factory|\Illuminate\View\View
         */
        public function shopCarDel($goods_id)
        {
            $user = Auth::user();
    
            if (!$user) {
                redirect()->route('home');
            }
    
            $goods = Redis::hexists('ShoppingCar :' . $user->id, $goods_id);
    
            if ($goods) {
                // 存在; 取出该商品数进行减1操作
                $num = Redis::hget('ShoppingCar :' . $user->id, $goods_id);
    
                if ($num > 1) {
                    $newNum = $num - 1;
                    Redis::hset('ShoppingCar :' . $user->id, $goods_id, $newNum);
                } else {
                    // 删除指定商品
                    Redis::hdel('ShoppingCar :' . $user->id, $goods_id);
                }
            }
    
            $allGoodsKey = Redis::hkeys('ShoppingCar :' . $user->id);
    
            if (null == $allGoodsKey) {
                $allGoodsNum  = false;
                $allGoodsInfo = false;
            }
    
            for ($i = 0; $i < count($allGoodsKey); $i++) {
                $allGoodsNum[$allGoodsKey[$i]]  = Redis::hget('ShoppingCar :' . $user->id, $allGoodsKey[$i]);
                $allGoodsInfo[$allGoodsKey[$i]] = json_decode(Redis::hget('GoodsInfo:Goods', $allGoodsKey[$i]), true);
            }
    
            return view('shopping_car.shopcar', [
                'allGoodsInfo' => $allGoodsInfo,
                'allGoodsNum'  => $allGoodsNum,
                'allGoodsKey'  => $allGoodsKey
            ]);
        }
    
        /**
         * 清空购物车
         * @param $goods_id
         * @return \Illuminate\Contracts\View\Factory|\Illuminate\View\View
         */
        public function cleanShopCar($goods_id)
        {
            $user = Auth::user();
            Redis::hdel('ShoppingCar :' . $user->id, $goods_id);
    
            $allGoodsKey = Redis::hkeys('ShoppingCar :' . $user->id);
    
            if (null == $allGoodsKey) {
                $allGoodsNum  = false;
                $allGoodsInfo = false;
            }
    
            for ($i = 0; $i < count($allGoodsKey); $i++) {
                $allGoodsNum[$allGoodsKey[$i]]  = Redis::hget('ShoppingCar :' . $user->id, $allGoodsKey[$i]);
                $allGoodsInfo[$allGoodsKey[$i]] = json_decode(Redis::hget('GoodsInfo:Goods', $allGoodsKey[$i]), true);
            }
    
            return view('shopping_car.shopcar', [
                'allGoodsInfo' => $allGoodsInfo,
                'allGoodsNum'  => $allGoodsNum,
                'allGoodsKey'  => $allGoodsKey
            ]);
        }
    }
```

### 使用到的`Redis`讲解

- **`Redis::exists()`**

> 检查给定 key 是否存在。
> > 若 key 存在，返回 1 ，否则返回 0 。

- **`Redis::hexists()`**

> 查看哈希表 `key` 中，给定域 `field` 是否存在。
> > 如果哈希表含有给定域，返回 1 。
    如果哈希表不含有给定域，或 key 不存在，返回 0 。

- **`Redis::hget()`**

> 返回哈希表 `key` 中给定域 `field` 的值。
> > 给定域的值。
    当给定域不存在或是给定 `key` 不存在时，返回 `nil` 。

- **`Redis::hset()`**

> 将哈希表 `key` 中的域 `field` 的值设为 `value` 。
  如果 `key` 不存在，一个新的哈希表被创建并进行 `HSET` 操作。
  如果域 `field` 已经存在于哈希表中，旧值将被覆盖。
> > 如果 `field` 是哈希表中的一个新建域，并且值设置成功，返回 `1` 。
    如果哈希表中域 `field` 已经存在且旧值已被新值覆盖，返回 `0` 。

- **`Redis::hmset()`**

> 同时将多个 `field-value` (域-值)对设置到哈希表 `key` 中。
  此命令会覆盖哈希表中已存在的域。
  如果 `key` 不存在，一个空哈希表被创建并执行 `HMSET` 操作。
> >如果命令执行成功，返回 `OK` 。
   当 `key` 不是哈希表(`hash`)类型时，返回一个错误。

- **`Redis::hkeys()`**

> 返回哈希表 `key` 中的所有域。
> > 一个包含哈希表中所有域的表。
    当 `key` 不存在时，返回一个空表。

- **`Redis::hdel()`**

> 删除哈希表 `key` 中的一个或多个指定域，不存在的域将被忽略。
> > 被成功移除的域的数量，不包括被忽略的域。

# 参考

- [Redis 命令参考](http://redisdoc.com/)

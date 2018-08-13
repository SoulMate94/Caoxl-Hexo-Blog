---
title: Laravel-admin-开发记录
date: 2018-01-04 15:47:18
categories: Laravel
tags: [Laravel, Laravel-admin, 框架, Framework]
---

# Laravel-admin
> laravel-admin 是一个可以快速帮你构建后台管理的工具
> 当前版本(1.5)需要安装PHP 7+和Laravel 5.5

<!--more-->

## 开发必备
- 文档: http://laravel-admin.org/docs/#/zh/
- Laravel文档: http://d.laravel-china.org/


## 安装/快速开始
- 安装
```php
composer require encore/laravel-admin "1.5.*"
```
- 发布资源：
```php
php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider"
```
- 完成安装：
```php
php artisan admin:install
```
- 添加路由器
```php
php artisan admin:make UserController --model=App\\User

// 在windows系统中
php artisan admin:make UserController --model=App\User
```
- 添加路由配置
```php
$router->resource('users', UserController::class);
```

## 帮助工具
- 脚手架
- artisan命令行工具
- 路由列表
- 文件管理
  - 安装
  ```php
  composer require laravel-admin-ext/media-manager -vvv
  php artisan admin:import media-manager
  ```
  - 配置
  ```php
    // 打开config/admin.php指定你要管理的disk
    'extensions' => [
        'media-manager' => [
            'disk' => 'public'   // 指向config/filesystem.php中设置的disk
        ],
    ],
  ```
- API tester
  - 安装
  ```php
    composer require laravel-admin-ext/api-tester -vvv
    php artisan vendor:publish --tag=api-tester
  ```
  - 导入菜单和权限
  ```php
    php artisan admin:import api-tester
  ```
- 配置管理
  - 安装
  ```php
   composer require laravel-admin-ext/config
   php artisan migrate
  ```
  - 打开app/Providers/AppServiceProvider.php, 在boot方法中添加Config::load();:
  ```php
    Config::load();  // 加上这一行
  ```
  - 导入菜单和权限
  ```php
   php artisan admin:import config
  ```
- 计划任务
  - 安装
  ```php
   composer require laravel-admin-ext/scheduling -vvv
   php artisan admin:import scheduling
  ```
  - 添加任务
  - 打开app/Console/Kernel.php， 试着添加两项计划任务：
  ```php
  class Kernel extends ConsoleKernel
  {
      protected function schedule(Schedule $schedule)
      {
          $schedule->command('inspire')->everyTenMinutes();
  
          $schedule->command('route:list')->dailyAt('02:00');
      }
  }
  ```

## 模型表格 Grid
```php
use App\Models\Movie;
use Encore\Admin\Grid;
use Encore\Admin\Facades\Admin;

$grid = Admin::grid(Movie::class, function(Grid $grid){

    // 第一列显示id字段，并将这一列设置为可排序列
    $grid->id('ID')->sortable();

    // 第二列显示title字段，由于title字段名和Grid对象的title方法冲突，所以用Grid的column()方法代替
    $grid->column('title');

    // 第三列显示director字段，通过display($callback)方法设置这一列的显示内容为users表中对应的用户名
    $grid->director()->display(function($userId) {
        return User::find($userId)->name;
    });

    // 第四列显示为describe字段
    $grid->describe();

    // 第五列显示为rate字段
    $grid->rate();

    // 第六列显示released字段，通过display($callback)方法来格式化显示输出
    $grid->released('上映?')->display(function ($released) {
        return $released ? '是' : '否';
    });

    // 下面为三个时间字段的列显示
    $grid->release_at();
    $grid->created_at();
    $grid->updated_at();

    // filter($callback)方法用来设置表格的简单搜索框
    $grid->filter(function ($filter) {

        // 设置created_at字段的范围查询
        $filter->between('created_at', 'Created Time')->datetime();
    });
});
```
- 禁用相关操作
```php
    禁用创建按钮
    $grid->disableCreation();
    
    禁用分页条
    $grid->disablePagination();
    
    禁用查询过滤器
    $grid->disableFilter();
    
    禁用导出数据按钮
    $grid->disableExport
    
    禁用行选择checkbox
    $grid->disableRowSelector();
    
    禁用行操作列
    $grid->disableActions();
    
    设置分页选择器选项
    $grid->perPages([10, 20, 30, 40, 50]);
```
### 内置方法:
- editable 直接编辑数据
```php
    $grid->title()->editable();
    $grid->title()->editable('textarea');
```
- switch
```php
    $grid->is_show('显示/隐藏')->switch(config('const.article_list.IS_SHOW'));
```
- switchGroup
```php
    $states = [
        'on' => ['text' => 'YES'],
        'off' => ['text' => 'NO'],
    ];
    
    $grid->column('switch_group')->switchGroup([
        'hot'       => '热门',
        'new'       => '最新'
        'recommend' => '推荐',
    ], $states);
```
### 帮助方法:
- 字符串操作
```php
    $grid->title()->limit(30)->ucfirst()->substr(1, 10);
```
- 数组操作
```php
    $grid->tags();
    // 调用Collection::pluck()方法取出数组的中的name列
    $grid->tags()->pluck('name');
    // 取出name列之后输出的还是数组，还能继续调用用Illuminate\Support\Collection的方法
    $grid->tags()->pluck('name')->map('ucwords');
    // 将数组输出为字符串
    $grid->tags()->pluck('name')->map('ucwords')->implode('-');
```
- 混合使用
```php
    // 链式方法调用来显示多图
    $grid->images()->display(function ($images) {
    
        return json_decode($images, true);
    
    })->map(function ($path) {
    
        return 'http://localhost/images/'. $path;
    
    })->image();
```
### 数据查询过滤
```php
$grid->filter(function($filter){

    // 去掉默认的id过滤器
    $filter->disableIdFilter();

    // 在这里添加字段过滤器
    $filter->like('name', 'name');
    ...

});
```
- 查询类型
  - $filter->equal('column', $label);
  - $filter->notEqual('column', $label);
  - $filter->like('column', $label);
  - $filter->ilike('column', $label);
  - $filter->gt('column', $label);
  - $filter->lt('column', $label);
  - $filter->between('column', $label);
  - $filter->between('column', $label)->datetime();//设置datetime类型
  - $filter->between('column', $label)->time(); //设置time类型
  - $filter->in('column', $label)->multipleSelect(['key' => 'value']);
  - $filter->notIn('column', $label)->multipleSelect(['key' => 'value']);
  - $filter->date('column', $label);
  - $filter->day('column', $label);
  - $filter->month('column', $label);
  - $filter->year('column', $label);

### 数据导出
- 安装
```php
    composer require maatwebsite/excel:~2.1.0
    php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```
- 新建自定义导出类，比如app/Admin/Extensions/ExcelExpoter.php:
```php
<?php

namespace App\Admin\Extensions;

use Encore\Admin\Grid\Exporters\AbstractExporter;
use Maatwebsite\Excel\Facades\Excel;

class ExcelExpoter extends AbstractExporter
{
    public function export()
    {
        Excel::create('Filename', function($excel) {

            $excel->sheet('Sheetname', function($sheet) {

                // 这段逻辑是从表格数据中取出需要导出的字段
                $rows = collect($this->getData())->map(function ($item) {
                    return array_only($item, ['id', 'title', 'content', 'rate', 'keywords']);
                });

                $sheet->rows($rows);

            });

        })->export('xls');
    }
}
```
- 在model-grid中使用这个导出类：
```php
    use App\Admin\Extensions\ExcelExpoter;
    
    $grid->exporter(new ExcelExpoter());
```


## 模型表单 Form
```php
use App\Models\Movie;
use Encore\Admin\Form;
use Encore\Admin\Facades\Admin;

$grid = Admin::form(Movie::class, function(Form $form){

    // 显示记录id
    $form->display('id', 'ID');

    // 添加text类型的input框
    $form->text('title', '电影标题');

    $directors = [
        'John'  => 1,
        'Smith' => 2,
        'Kate'  => 3,
    ];

    $form->select('director', '导演')->options($directors);

    // 添加describe的textarea输入框
    $form->textarea('describe', '简介');

    // 数字输入框
    $form->number('rate', '打分');

    // 添加开关操作
    $form->switch('released', '发布？');

    // 添加日期时间选择框
    $form->dateTime('release_at', '发布时间');

    // 两个时间显示
    $form->display('created_at', '创建时间');
    $form->display('updated_at', '修改时间');
});

```

### 自定义工具
```
$form->tools(function (Form\Tools $tools) {

    // 去掉返回按钮
    $tools->disableBackButton();

    // 去掉跳转列表按钮
    $tools->disableListButton();

    // 添加一个按钮, 参数可以是字符串, 或者实现了Renderable或Htmlable接口的对象实例
    $tools->add('<a class="btn btn-sm btn-danger"><i class="fa fa-trash"></i>&nbsp;&nbsp;delete</a>');
});
```

### 其它方法
- 去掉提交按钮:
```php
    $form->disableSubmit();
```
- 去掉重置按钮:
```php
    $form->disableReset();
```
- 忽略掉不需要保存的字段
```php
    $form->ignore(['column1', 'column2', 'column3']);
```
- 设置宽度
```php
    $form->setWidth(10, 2);
```
- 设置表单提交的action
```php
    $form->setAction('admin/users');
```

## 权限控制
### 相关方法:
- 获取当前用户对象
```php
    Admin::user();
```
- 获取当前用户id
```php
    Admin::user()->id;
```
- 获取用户角色
```php
    Admin::user()->roles;
```
- 获取用户的权限
```
    Admin::user()->permissions;
```
- 用户是否某个角色
```
    Admin::user()->isRole('developer');
```
- 是否有某个权限
```
    Admin::user()->can('create-post');
```
- 是否没有某个权限
```
Admin::user()->cannot('delete-post');
```
- 是否是超级管理员
```
    Admin::user()->isAdministrator();
```
- 是否是其中的角色
```
Admin::user()->inRoles(['editor', 'developer']);
```

### 权限中间件:

```
    // 允许administrator、editor两个角色访问group里面的路由
    Route::group([
        'middleware' => 'admin.permission:allow,administrator,editor',
    ], function ($router) {
    
        $router->resource('users', UserController::class);
        ...
    
    });
    
    // 禁止developer、operator两个角色访问group里面的路由
    Route::group([
        'middleware' => 'admin.permission:deny,developer,operator',
    ], function ($router) {
    
        $router->resource('users', UserController::class);
        ...
    
    });
    
    // 有edit-post、create-post、delete-post三个权限的用户可以访问group里面的路由
    Route::group([
        'middleware' => 'admin.permission:check,edit-post,create-post,delete-post',
    ], function ($router) {
    
        $router->resource('posts', PostController::class);
        ...
    
    });
```


## 自定义
- 自定义登录认证
  - http://laravel-admin.org/docs/#/zh/custom-authentication
- 自定义头部导航条
  - http://laravel-admin.org/docs/#/zh/custom-navbar
- 自定义图表
  - http://laravel-admin.org/docs/#/zh/custom-chart
  

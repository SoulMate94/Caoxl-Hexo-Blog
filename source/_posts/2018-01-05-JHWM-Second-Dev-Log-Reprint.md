---
title: 二次开发之江湖外卖-转
date: 2018-01-05 15:15:15
categories: Framework
tags: [PHP, Framework]
---

## 惠吃猫-二次开发-江湖外卖
> 二次开发一个买来的项目,源码相当垃圾,没有文档

<!--more-->
 
### 安装&开发配置

- 需要的扩展
  - ZendGuardLoader
  - Redis
- 设置数据库连接：system/config.php
- 自定义绑定域名：/admin(后台) => 设置 => 网站设置 => 基本设置 => 网站网址


### 通过 SSH tunnel(隧道) 连接阿里云 RDS(关系数据系统)
首先要要可以连接到一台阿里云主机
- 然后先设置 SSH 连接（连ECS）
  - ssh root@ECS公网IP
- 最后设置数据库连接（连 RDS）

### PHPStorm
 - 本地上传远程
   - 先打开项目再配置远程
     - 配置远程
     - 本地使用 PHPStorm 打开 git 仓库后，选中项目名称，右键 “upload to xxx” 即可（此操作最好只在最开始操作一次，因为遍历所有文件会比较慢）
     - 可以设置一直 "upload to xxx"
   - 先配置远程再打开项目
   
## 系统
 - __CORE_DIR：system 目录在文件系统中的绝对路径。如：/data/wwwroot/jhwm.dev/system/
 - __TIME：时间戳
 - EOOTURL：网站根URL。如：http://jhwm.dev/。
 - $pager 结构
 ```php
     {
       "page":1,
       "limit":50,
       "count":11,
       "pagebar":null,
       "res":"\ static",
       "img":"http://jhwm.dev/attachs",
       "dateline":1495853330,
       "url":"http://admin"
     }
 ```
 可以在后台：设置=》网站设置=》附件设置中修改”附件URL地址”。
 - 伪静态开启：设置 => 基本设置 => 开启伪静态。

### 自定义配置
如果需要在后台新增一个选项，需要先往数据库 system_module 中插入一条记录。比如：

```
insert into jh_system_module values(null, 'module', 3, 'system/conf', 'index', '系统配置', 1, 269, 50, current_timestamp);
insert into jh_system_module values(null, 'module', 3, 'system/conf', 'update', '系统配置', 0, 269, 50, current_timestamp);
```

其中 visible 字段表示时候显示到后台页面。

注意，只要是要通过 HTTP 请求系统配置的，都需要插入记录先，但是在相应的配置控制器中，则不需要为没个方法都插入一条记录。

## 接口
统一请求：/api.php，POST 发送数据类型为 json。示例：

```php
http://jhwm.dev/api.php?API=shop/items&data={%22lat%22:%2237.7914833290724%22,%22lng%22:%22-122.4000111082433%22,%22page%22:1}
```
其中，该 URL 最终访问到的控制器和方法是：system/api/controllers/shop.ctl.php@items()。
- 如何写一个接口

```php
    // 数据库查询、数据格式组装（如果需要）
    // 返回
    $this->msgbox->add('success');
    $this->msgbox->set_data('data', []);
```

### 接口开发
- iOS／Android 如何从 H5 跳回 APP？
有时候，需要从 APP 内跳入 H5 页面进行一些操作，比如支付。支付结束不希望继续留在 Web 页面而是返回 APP，这样的体验会好很多。

可以在返回的页面带上一个特殊的字符串参数，比如 _app_listen_on_HASHSTRING。然后 APP 内可以从对 URL 请求中监听到这个特殊的字符串，然后自行返回到 APP 内部。

## 数据库
- 打印 SQL（只有通过日志的形式；只能打印全部的）

```php
print_r($this->system->db->SQLLOG());
K::M("system/logs")->log("sqllog",$this->system->db->SQLLOG());
```

- iOS 调用 payment/moneydeduction 接口时候 把拿到的参数调用支付宝APP 提示 “创建交易异常，请重新创建后再付款”（web 支付可以）
  - wap端和app用的不是同一种加密方式
  - app的支付 一般和公私钥的配置有关系（mapi网关密钥支付）
  - 公私钥 配置正确
  
### 模型
 - 表单命名和数据库表字段命名要一致，并在 protected $_cols 里面把可以新增和修改的字段添加进去。示例：system/models/shop/shop.mdl.php
 - K::M('<ModelCate>/<Model>') 可以创建任何位于 system/models 目录下的模型。

### 自定义数据库连接
再使用的过程中，发现很多对数据库的操作都没有，也无文档可查，一“怒”之下自己封装了一个简单的 PDO 操作类，供自己使用：

```php
    <?php
    // system/models/system/dbext.mdl.php
    if(!defined('__CORE_DIR')){
        exit("Access Denied");
    }
    class Mdl_System_Dbext extends Model
    {
        public $pdo = null;
        public $tbPre = '';
        public function __construct()
        {
            unset($this->system->db);
            if (!$this->pdo || !is_object($this->pdo)) {
                $this->pdo = $this->getDbInstance();
            }
        }
        public function getDbInstance()
        {
            if (!$this->pdo || !is_object($this->pdo)) {
                $dbCfg  = parse_url(__CFG::MYSQL);
                $path   = array_filter(explode('/', $dbCfg['path']));
                $dbName = isset($path[1]) ? ';dbname='.$path[1] : '';
                $chSet  = isset($path[3]) ? $path[3] : 'UTF8';
                $chSet  = ';charset='.$chSet;
                if (isset($path[2])) {
                    $this->tbPre = $path[2];
                }
                $dsn    = $dbCfg['scheme'].':host='.$dbCfg['host'].$chSet.$dbName;
                $user   = $dbCfg['user'];
                $passwd = $dbCfg['pass'];
                return new PDO($dsn, $user, $passwd);
            } else {
                return $this->pdo;
            }
        }
    }
```

## 系统成员

### 用户
 - 在接口中是 $this->member 在 web 中是 $this->user。
### 商户
 - 登录后可以通过 $this->shop 获取信息。
### API
 - 为什么请求 /api.php?API=biz/info&TOKEN=XXXX 带上 TOKEN 了还是要求登录？
 源码：system/api/controller.php@check_login() 中可以看出，必须还要带上 CLIENT_API=BIZ 才可以。配送端同理。
### 商户后台
 - system/home/controllers/biz/ctlmaps.php 这里事先配置好菜单及其子项才可以
 
## 常用调用
 - 创建一个连接
 
 ```php
     K::M('helper/link')->mklink('trade/payment:return', array('alipay'), null, 'www');
     // http://jhwm.dev/index.php?trade/payment/return-alipay.html
     // app 和 www 的区别只是有没有 http://jhwm.dev
 ```

## 注意事项（坑）
 - web 登录一个账户后，如果再使用 api 登录，则会清除登录信息。
 - 后台的配置信息时报存在数据库的。

## FAQ

### 启用阿里云全站 CDN 后，访问 /admin 会出现无限 302？
检查是否在 CDN 设置中开启了「过滤参数」 选项（性能优化分类下）。
> 回源时会去除 URL 中？之后的参数，有效提高文件缓存命中率，提升分发效率。

而江湖外卖对请求的处理是根据请求参数来选择控制器和方法的。以登录后台为例，江湖外卖的源代码又如下判断：

```php
    // system/admin/index.php
    protected function _init()
    {
      $guest_allow = array("index:login", "index:verify", "index:loginout");
      if ($OATOKEN = trim($_POST["OATOKEN"])) {
        if ($a = $this->load_model("secure/crypt")->hexarr($OATOKEN)) {
          if ($a["ATOKEN"] && $a["AGENT"]) {
            $_SERVER["HTTP_USER_AGENT"] = $a["AGENT"];
            $_COOKIE[__CFG::C_PREFIX . "ATOKEN"] = $a["ATOKEN"];
          }
        }
      }
      define("CITY_ID", $this->admin->admin["city_id"]);
      parent::_init();
      require __APP_DIR . "controller.php";
      $act = $this->request["ctl"] . ":" . $this->request["act"];
      $this->admin = K::M("admin/auth");
      // ---- 问题代码所在 开始 ----
      if (!$this->admin->token()) {
        if (!in_array($act, $guest_allow)) {
          header("Location:?index-login");
          exit();
        }
      }
      // ---- 问题代码所在 结束 ----
      
      $this->admin_id = $this->admin->admin_id;
      $this->admin_name = $this->admin->admin_name;
    }
```

可见，如果开起来过滤参数，那么 CDN 对动态请求回源时会经过如下流程：
 1. 用户访问 /admin
 2. 阿里云全站 CDN 识别是动态请求，在过滤掉请求参数后回源
 3. 源站（江湖外卖）检查出未登录，将重定向到登录界面：/admin?index-login
 4. CDN 接收到重定向请求，会把 /admin?index-login 中的参数过滤变成 /admin，然后重新回源请求
 5. 源站（江湖外卖）检查出未登录，将重定向到登录界面：/admin?index-login
 6. CDN 接收到重定向请求，会把 /admin?index-login 中的参数过滤变成 /admin，然后重新回源请求
 7. ...
 
因此，对于江湖外卖这套系统，在启用 CDN 的时候，不要启用参数过滤。

> 实际上，很多系统在开启 CDN 时候如果启用了性能优化都会出现一些问题，因为 CDN 提供的性能优化对很多系统是不适用的。

### 启用阿里云 CDN 后，后台进入商家管理信息读取异常？
具体表现是：从管理总后台商家列表点击进入商家管理后台时，随机切换左侧标签页，显示的商家名称都不一样。

问题的原因是，江湖外卖在对动态请求的响应头中，并没有设置 no-cache，导致 CDN 缓存了 cookie，而 cookie 是要被用于源站。

> - 请务必将 Cache-Control 设置为no-cache, private或者max-age=0。（动态文件一般类似是带有cookie id 的登陆页面，交易页面，或者是需要与数据库进行交互生成的页面）, 这样CDN就不会做缓存，直接回源站；
> - 如果加速域名下面的文件类型多为动态文件，强烈建议采用独立域名，不用CDN加速

## 附录
 - POST JSON data with CURL

 ```php
    curl -X POST http://localhost/api/login \
    -H "Content-Type: application/json"
    -d '{"username":"xyz","password":"xyz"}'
 ```
 
## 参考
 - [缓存相关-阿里云CDN帮助文档](https://help.aliyun.com/document_detail/27265.html)
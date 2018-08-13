---
title: 七牛云 「开发日志」
date: 2018-01-23 11:25:02
categories: ThirdParty
tags: [Qiniu, 七牛, 第三方, ThirdParty]
---

> 总结下使用七牛云 PHP SDK 和 JS SDK 的过程中经常使用的代码 demo，以及坑。

# PHP-SDk

## 安装

```
    composer config -g repo.packagist composer https://packagist.phpcomposer.com
    
    composer require qiniu/php-sdk
```

<!--more--> 

## 操作的前提和对象

- 「前提」就是认证 acess_key 和 secret_key；
- 「对象」就是认证通过后的 $auth 对象。

在调用 PHP SDK 的时候，所有七牛云的操作都是通过这个通过认证的 $auth 对象来完成的。
    
```
    use Qiniu\Auth;
    
    $ak = env('QINIU_ACCESSKEY');
    $sk = env('QINIU_SECRETKEY');
    
    $auth = new Auth($ak, $sk);
```
简单地说，**在 SDK 中调用七牛云的所有 API 都需要从获得 $auth 对象开始**。

## 私有空间

### 获得私有空间的下载链接

想要操作私有空间的文件，首先必须对私有空间文件进行签名，然后把签名的结果当作公开文件外链那样再进行所需处理。

```
    # 私有空间文件地址 = 私有空间域名+私有空间中的文件 key
    $prvt_url = 'http://prvt.eme168.com/00521276847c7dfc21981faaddb1881e14766870526616.jpg?attname='.$save_as;
        
    // $save_as 是下载后的文件名 可以不选
    echo $auth->privateDownloadUrl($prvt_url, 600);    // 600 指的是该下载链接在公网上的失效时间
```

### 下载链接=访问链接

简单的说，七牛云中私有空间的下载链接可以当做公开空间的外链使用。

这其实也是公共空间和私有空间调用七牛云 API 的主要区别。

下载链接要真正返回一个下载相应的话，请求参数中带上 `?attname=SAVE_AS` 即可。

*注意*

对于中文文件名，为了更好的支持中文，将 `attname=` 后面的中文文件名部分要进行 `URL encode` 操作。

## 持久化

七牛云中的「持久化（pfop）」就是用队列的形式处理一系列耗时操作。

### 上传时触发

只需在构造上传凭证时，在上传策略中设置 `persistentOps` 和 `persistentNotifyUrl` 两个字段。

**举例说明:**

```
    $bucket     = 'private';
    $origin_img = '00521276847c7dfc21981faaddb1881e14766870526616.jpg';
    $notify_url = 'http://www.eme168.com/api/cloudcapture/shrink_notify';
    
    $policy = [
      'persistentOps'       => $pfop,
      'persistentNotifyUrl' => $notify_url    // 必须和 persistentOps 同时存在
    ];
    
    $token = $auth->uploadToken(
      $bucket, null, 7200, $policy
    );
    
    $upload_res = $uploader->putFile(
      $token,        // 上传 token
      $origin_img,   // 上传后要重命名时需填 不需要时直接填 null 即可
      base_path($save_path).'/'.$origin_img    // 要上传文件的本地路径
    );
```

### pipeline

- 持久化操作期间使用的队列名称（pipeline）可选 可以不填或填 null 但不能乱填。举例说明：

```
    use Qiniu\Processing\PersistentFop;
    
    $bucket   = 'private';     // 资源空间名
    $pipeline = 'em-shrink-'.rand(1, 16);    // 或者 $pipeline = null
    $pfop = new PersistentFop($auth, $bucket, $pipeline);
```

当 `$pipeline` 为 null 时，七牛云会默认为你的这个操作分配一个队列名。

- `pipeline` 需要事先在控制台创建

### 保存持久化处理结果

```
    $origin_img   = '00521276847c7dfc21981faaddb1881e14766870526616';
    $shrink_name  = 'shrink_'.(microtime(true)*10000).'.jpg';
    $save_as      = base64_encode('yimai:'.$shrink_name);
    $fops = 'imageMogr2/thumbnail/!30p/crop/x500|saveas/'.$save_as;
    list($id, $err) = $pfop->execute($origin_img, $fops);    // $pfop 是持久化操作对象
    
    echo "\n====> pfop thumbnail result: \n";
    if ($err != null)
      print_r($err);
    else
      echo "PersistentFop Id: $id\n";
```

### 持久化处理通知回调

持久化处理本质是对七牛云一些列 API 的请求，七牛云在自己的队列中处理，处理后通过在构造上传 token 时指定的 persistentNotifyUrl 来主动通知到业务服务器。

```
    // 处理七牛云持久化操作结束后的通知请求
    public function prstNotifyCallBack(CloudCaptureModel $ccm)
    {
      $notify_body = file_get_contents('php://input');
      $this->log('prstNotifyCallBack', $notify_body);
      if ($notify_body) {
        $data = json_decode($notify_body, true);
        if ($data['code']===0 && $data['items']) {
          // 你的业务代码
        }
      } else exit('empty php input flow');
    }
```

### 查询持久化处理结果

#### 通过持久化处理 ID

```
    http://api.qiniu.com/status/get/prefop?id=<PersistentFop Id>    # z0.582b573845a2655db7679b5f
```

#### 通过 PHP SDK

```
    list($ret, $err) = $pfop->status($id);    // $pfop 是持久化操作对象
    
    echo "\n====> pfop xxxx status: \n";
    
    if ($err != null)
      print_r($err);  
    else
      print_r($ret);
```

## 下载

### 单文件下载

> 「公开」和「私有」下载链接的区别只是签名与否。

#### 客户端

```
    $.get('/get_down_link', {
    	'id' : 1
    }, function (res) {
    	if (res.status != '500') {
        	saveFile(res.link);
    	}
    });
    
    function saveFile(down_link){
        var a  = document.createElement('a');
        a.href = down_link;
        a.download = down_link;
        a.click();
    };
```

#### 服务器

```
    $this->validate($req, [
      'id' => 'required|integer|min:1'
    ]);
    $id = $req->get('id');
    // get the file key of this id in qiniu from db => $data
      	
    if ($data) {
      $uri = 'http://www.proj.com/'
      .$data['origin']
      .'?attname='
      .$save_as;    // 下载时保存到本地的文件名（中文名等请强制使用 `urlencode()` 编码）
      // if store in private buket (assume here)
      $link = $private
      ? $auth->privateDownloadUrl($uri, 7200)
      : $uri;
      return [
          'status' => 200,
          'link'   => $link
      ];    // auto json response
    }
    return ['status' => 500];
```

### 多文件打包下载

当出现批量下载的需求的时候，就得先调用 pfop 处理成一个压缩包，然后再走单文件下载的思路就行了。

```
    public $qiniu_prst_query_api = 'http://api.qiniu.com/status/get/prefop?id=';
    protected $origin_buket      = 'private';
    
    // yield the compress file with the res origin keys
    // Array $origins
    public function yieldQiniuCompressFile($origins)
    {
      if (is_array($origins) and $origins) {
        $ak   = env('QINIU_ACCESSKEY');
        $sk   = env('QINIU_SECRETKEY');
        $auth = new Auth($ak, $sk);
        
        // 指定 buket 中存在的 要压缩的文件名
        $key = $origins[0]['origin'];
        
        $pfop = new PersistentFop($auth, $this->origin_buket);
        
        // 压缩后的文件名
        $zip_key = date('YmdHis').'.zip';
        
        $fops = 'mkzip/2';
        
        // 需要进行 zip 压缩的文件名 uri（包含第一个 $key 指定的文件）
        foreach ($origins as $k => $val) {
          $org_url  = $this->prvt_domain
          .$val['cc_origin'].'?attname='.$val['name'].'.'.$val['format'];
          $prvr_url = $auth->privateDownloadUrl($org_url, 3600);
          $fops .= '/url/'  .\Qiniu\base64_urlSafeEncode($prvr_url);
          $fops .= '/alias/'.\Qiniu\base64_urlSafeEncode(
            $val['name'].'_'.$k.'.'.$val['format']
          );
        }
        
        // 指定压缩文件保存的路径
        $fops .= '|saveas/' . \Qiniu\base64_urlSafeEncode($this->origin_buket.':'.$zip_key);
        list($id, $err) = $pfop->execute($key, $fops);
        if (!$err) {
        	return [
            	'prstId' => $id,
            	'resUrl' => $this->qiniu_prst_query_api.$id
            ];
        }
      }
      
      return 500;
    }
    // query persistent process res status
    public function getQiniuPrstStatus(Request $req)
    {
      $this->validate($req, [
        'resUrl' => 'required'
      ]);
      
      $ch = curl_init();
      curl_setopt($ch, CURLOPT_URL, $req->get('resUrl'));
      curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
      $res = curl_exec($ch);
      
      return curl_errno($ch)
        ? 500
        : $res;
    }
```

唯一需要注意的一点是，上面的 `key = $origins[0]['origin']` 仅仅为符合 `pfop` 操作的接口规格而存在，并没有实际的意义，但需要是操作空间中存在的资源的 `key`。（其实我是有点费解为啥要把接口这样定义绕）


# JS SDK

## 上传&回调

```
    bower install qiniu
```

然后在服务器上新建一个 qiniu.html，内容如下：

```
    <!DOCTYPE html>
    <html lang="en">
    <head>
    	<meta charset="UTF-8">
    	<title>qiniu test</title>
    </head>
    <body>
    <div id="container">
        <a class="btn btn-default btn-lg " id="pickfiles" href="#" >
            <i class="glyphicon glyphicon-plus"></i>
            <span>选择文件</span>
        </a>
    </div>
    <input type="hidden" id="upToken">
    <script src="/js/jquery.min.js"></script>
    <script src="/bower_components/plupload/js/moxie.js"></script>
    <script src="/bower_components/plupload/js/plupload.dev.js"></script>
    <script src="/bower_components/qiniu/dist/qiniu.js"></script>
    
    <script>
    var uploader = Qiniu.uploader({
        runtimes: 'html5,flash,html4',      // 上传模式，依次退化
        browse_button: 'pickfiles',         // 上传选择的点选按钮，必需
        // 在初始化时，uptoken，uptoken_url，uptoken_func三个参数中必须有一个被设置
        // 切如果提供了多个，其优先级为uptoken > uptoken_url > uptoken_func
        // 其中uptoken是直接提供上传凭证，uptoken_url是提供了获取上传凭证的地址，如果需要定制获取uptoken的过程则可以设置uptoken_func
        // uptoken : '', // uptoken是上传凭证，由其他程序生成
        // uptoken_url: 'http://api.example.com/qiniu/up_ticket',         // Ajax请求uptoken的Url，强烈建议设置（服务端提供）
        uptoken_func: function(){    // 在需要获取uptoken时，该方法会被调用
        	var jqxhr = $.ajax({
        	   url: 'http://api.example.com/qiniu/up_ticket',
        	   type: 'GET',
        	   dataType: 'json',
        	   async: false,
        	   cors: true ,
        	   headers: {
        			'Access-Control-Allow-Origin': '*',
        			'AUTHORIZATION': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1aWQiOjF9.SscwZjqXAkp1gY3eVBiMBpdtQhyfcV7V67UDNeacJhc'
        	   },
        	   success: function (res) {
        	   },
        	   error: function (xhr, status) {
        	   }
        	});
        	
        	return jqxhr.responseJSON.upToken;
        },
        get_new_uptoken: false,             // 设置上传文件的时候是否每次都重新获取新的uptoken
        // downtoken_url: '/downtoken',
        // Ajax请求downToken的Url，私有空间时使用，JS-SDK将向该地址POST文件的key和domain，服务端返回的JSON必须包含url字段，url值为该文件的下载地址
        unique_names: true,              // 默认false，key为文件名。若开启该选项，JS-SDK会为每个文件自动生成key（文件名）
        // save_key: true,                  // 默认false。若在服务端生成uptoken的上传策略中指定了sava_key，则开启，SDK在前端将不对key进行任何处理
        domain: 'static.hcmchi.com',     // bucket域名，下载资源时用到，必需
        container: 'container',             // 上传区域DOM ID，默认是browser_button的父元素
        max_file_size: '100mb',             // 最大文件体积限制
        flash_swf_url: '/bower_components/plupload/js/Moxie.swf',  //引入flash，相对路径
        max_retries: 3,                     // 上传失败最大重试次数
        dragdrop: true,                     // 开启可拖曳上传
        drop_element: 'container',          // 拖曳上传区域元素的ID，拖曳文件或文件夹后可触发上传
        chunk_size: '4mb',                  // 分块上传时，每块的体积
        auto_start: true,                   // 选择文件后自动上传，若关闭需要自己绑定事件触发上传
        //x_vars : {
        //    查看自定义变量
        //    'time' : function(up,file) {
        //        var time = (new Date()).getTime();
                  // do something with 'time'
        //        return time;
        //    },
        //    'size' : function(up,file) {
        //        var size = file.size;
                  // do something with 'size'
        //        return size;
        //    }
        //},
        init: {
            'FilesAdded': function(up, files) {
                plupload.each(files, function(file) {
                    // 文件添加进队列后，处理相关的事情
                });
            },
            'BeforeUpload': function(up, file) {
                   // 每个文件上传前，处理相关的事情
            },
            'UploadProgress': function(up, file) {
                   // 每个文件上传时，处理相关的事情
            },
            'FileUploaded': function(up, file, info) {
                   // 每个文件上传成功后，处理相关的事情
                   // 其中info是文件上传成功后，服务端返回的json，形式如：
                   // {
                   //    "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
                   //    "key": "gogopher.jpg"
                   //  }
                   // 查看简单反馈
                   // var domain = up.getOption('domain');
                   // var res = parseJSON(info);
                   // var sourceLink = domain +"/"+ res.key; 获取上传成功后的文件的Url
            },
            'Error': function(up, err, errTip) {
                   //上传出错时，处理相关的事情
            },
            'UploadComplete': function() {
                   //队列文件处理完毕后，处理相关的事情
            },
            'Key': function(up, file) {
                // 若想在前端对每个文件的key进行个性化处理，可以配置该函数
                // 该配置必须要在unique_names: false，save_key: false时才生效
                var key = "";
                // do something with key here
                return key
            }
        }
    });
    </script>
    	
    </body>
    </html>
```

## JS-SDK 上传完成后七牛云回调参数中没有文件名

需要在 JS-SDK 获取上传凭证的时候定制一下 `callbackBody` 的格式：

```
    $policy = [
      'callbackUrl'  => route('qiniu_upload_callback'),
      
      # 1. 通过 JSON 格式
      'callbackBody' => json_encode([
        // 'name'  => '$(fname)',
        // 'hash'  => '$(etag)',
        'fkey'  => '$(key)',
        'us_id' => $this->req->us_id,
        'id'    => $this->req->id,
      ]),
      
      # 2. 通过 URL 参数对形式
      // 'callbackBody' => 'fkey=$(key)&us_id='.$this->req->us_id.'&id='.$this->req->id,
    ];
```

然后再回调中解析出来：

```
    $callbackBody = file_get_contents('php://input');
```

注意回调这里收到的参数格式和上传策略（`$policy`）中定义的 `callbackBody` 是一样的，因此如果是使用 URL 参数的形式传过去的，接受的时候就不要用 `json_decode `来解析了。


## 三种方式指定上传后的文件名

- JS-SDK 中指定 `unique_names` 为 true，由 JS-SDK 指定。
- 上传策略中指定 `callbackFetchKey`，然后在七牛回调应用服务器的时候由应用服务端指定。
- 服务端生成上传凭证的时候，在上传策略中指定 `save_key`，该 key 就是最终上传的文件名。

## 多个 uploader 返回值不一致问题

png/jpg 和 rar/zip 格式的文件上传成功后，返回值不一致。

 - `png/jpg` 上传成功后其 key 为 `JSON.parse(info.response).key` ，
 - 而 `rar/zip` 上传成功后其 key 在 `file.target_name` 中。


# 官方 API

## 接口连贯操作

### 同处理接口下的连贯操作

```
    # 通过 K/V
    http://static.eme168.com/12c8da59f1203ccc99beedd31d9dd02214759391349863.jpg?imageMogr2/thumbnail/!80p/crop/x300
```

### 不同处理接口下的连贯操作

```
    # 通过管道符 |
    http://static.eme168.com/12c8da59f1203ccc99beedd31d9dd02214759391349863.jpg?imageMogr2/thumbnail/!30p|imageMogr2/crop/x300
```

### 和 PHP SDK 一致

上面 URL 后面的参数，比如 `imageMogr2/thumbnail/!80p/crop/x300`，这个和在 PHP SDK 中触发持久化时传递的操作名是一致的。

# 可能遇到的坑

## 获取含有空格文件名的图片信息

这个坑在于，当使用 `<download_url?imageInfo>` 在浏览器实时测试时是发现不了的，因为浏览器（Chrome）默认已经处理过了，只有使用 `SDK`（PHP-SDK）时才能发现，这时候需要按照浏览器自动转换的格式，**把空格编码成 %20 **即可。

```
    $file = 'abcd 中文.png';
    $uri  = rawurlencode($file).'?imageInfo';    -- not `urlencode()`
```

# FAQ

- PHP-SDK 上传文件时触发持久化操作，在文件大于 20M 后持久化操作失败

出现此情况的原始持久化操作为：`imageslim|imageMogr2/crop/x1170`。这是七牛暂时没有处理好的地方。

解决办法是，连个持久化顺序调换：`imageMogr2/crop/x1170|imageslim`。原因是：

> 这种方式 认为 imageslim 是异步，但是 imageMogr2 是同步了，导致 imageMogr2 处理失败了。
> 
> 原本是同步处理的 imageMogr2 操作会不允许大于 20M 的文件直接处理，会允许异步的 imageMogr2 操作，这里看起来应该是系统将管道中的处理 也认为是 同步处理了。
> 
> 换个顺序应该是 imageMogr2 被认为是异步的，大于 20M 可以处理，处理的结果应该会小于 20M，所以 imageslim 即使是同步也可以被处理成功。


# 参考

- [七牛开发者中心](https://developer.qiniu.com/kodo)
- [处理结果另存(saveas)](https://developer.qiniu.com/dora/manual/1305/processing-results-save-saveas)
- [上传文件七牛回调时没有传回文件名](https://segmentfault.com/q/1010000003890918)


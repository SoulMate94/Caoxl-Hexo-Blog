---
title: 「代码复用」PHP常用代码
date: 2018-01-23 17:16:54
categories: CodeReuse
tags: [PHP, Code]
---

> 记录一些PHP常用方法/代码
> > 以下方法 个人收集 转载做业务需求与本人无关~

<!--more-->

# 服务器相关

## 获取服务器IP地址

### PHP获取服务器IP地址方法一：

```
    if('/'==DIRECTORY_SEPARATOR){
        $server_ip=$_SERVER['SERVER_ADDR'];
    }else{
        $server_ip=@gethostbyname($_SERVER['SERVER_NAME']);
    }
    
    echo $server_ip;
```

### php获取服务器ip地址方法二：

```
    /**
    * 获取服务器端IP地址
     * @return string
    */
     
    function get_server_ip(){
        if(isset($_SERVER)){
            if($_SERVER['SERVER_ADDR']){
                $server_ip=$_SERVER['SERVER_ADDR'];
            }else{
                $server_ip=$_SERVER['LOCAL_ADDR'];
            }
        }else{
            $server_ip = getenv('SERVER_ADDR');
        }
        return $server_ip;
    }
     
    echo get_server_ip();
```

### 获取访问用户的IP

```
    private  function getAccessUserIP()
    {
        if (getenv("HTTP_X_FORWARDED_FOR")) {
            //这个提到最前面，作为优先级,nginx代理会获取到用户真实ip,发在这个环境变量上，必须要nginx配置这个环境变量HTTP_X_FORWARDED_FOR
            $ip = getenv("HTTP_X_FORWARDED_FOR");
        } else if (getenv("REMOTE_ADDR")) {
            //在nginx作为反向代理的架构中，使用REMOTE_ADDR拿到的将会是反向代理的的ip，即拿到是nginx服务器的ip地址。往往表现是一个内网ip。
            $ip = getenv("REMOTE_ADDR");
        } else if ($_SERVER['REMOTE_ADDR']) {
            $ip = $_SERVER['REMOTE_ADDR'];
        } else if (getenv("HTTP_CLIENT_IP")) {
            //HTTP_CLIENT_IP攻击者可以伪造一个这样的头部信息，导致获取的是攻击者随意设置的ip地址。
            $ip = getenv("HTTP_CLIENT_IP");
        } else {
            $ip = "unknown";
        }
        return $ip;
    }
```

## 返回ip所在的区域 外国ip精确到国名

```
    public function checkAccessIP()
    {
        // 根据IP判断是否显示购买
        header('Content-Type:text/html;charset=utf-8');
        $AccessIP = $this->getAccessIP();   // 获取访问者IP地址
        $CountryName = $this->getPosition($AccessIP);  // 返回访问IP国家名称
        //$notAllowArea = ['美国','加拿大','日本'];
        $AllowArea = ['中国'];
        if (in_array($CountryName,$AllowArea)){
            $data = array(
                'isShowButton' => '1',
                'msg'  => '允许访问',
            );
            echo json_encode($data,JSON_UNESCAPED_UNICODE);exit();
        } else {
            $data = array(
                'isShowButton' => '0',
                'msg'  => '禁止访问',
            );
            echo json_encode($data,JSON_UNESCAPED_UNICODE);exit();
        }
    }
    
    //返回ip所在的区域 外国ip精确到国名
    protected function getPosition($ip){
        try {
            $res = file_get_contents("http://ip.taobao.com/service/getIpInfo.php?ip=$ip");
            $res = json_decode($res,true);
    
            if ($res[ "code"]==0){
                //return $res['data']["country"].$res1['data'][ "region"].$res1['data']["city"]."_".$res1['data'][ "isp"];
                return $res['data']["country"];
            }else{
                return "未能获取";
            }
        }catch (Exception $e){
            return "未能获取";
        }
    }
    
    private  function getAccessIP()
    {
        if (getenv("HTTP_X_FORWARDED_FOR")) {
            //这个提到最前面，作为优先级,nginx代理会获取到用户真实ip,发在这个环境变量上，必须要nginx配置这个环境变量HTTP_X_FORWARDED_FOR
            $ip = getenv("HTTP_X_FORWARDED_FOR");
        } else if (getenv("REMOTE_ADDR")) {
            //在nginx作为反向代理的架构中，使用REMOTE_ADDR拿到的将会是反向代理的的ip，即拿到是nginx服务器的ip地址。往往表现是一个内网ip。
            $ip = getenv("REMOTE_ADDR");
        } else if ($_SERVER['REMOTE_ADDR']) {
            $ip = $_SERVER['REMOTE_ADDR'];
        } else if (getenv("HTTP_CLIENT_IP")) {
            //HTTP_CLIENT_IP攻击者可以伪造一个这样的头部信息，导致获取的是攻击者随意设置的ip地址。
            $ip = getenv("HTTP_CLIENT_IP");
        } else {
            $ip = "unknown";
        }
        return $ip;
    }
```

## 获取当前的域名

```
      <?php
        
      //获取当前的域名:  
      echo $_SERVER['SERVER_NAME'];  
      
      //获取来源网址,即点击来到本页的上页网址  
      echo $_SERVER["HTTP_REFERER"];
        
      $_SERVER['REQUEST_URI'];//获取当前域名的后缀  
      $_SERVER['HTTP_HOST'];//获取当前域名  
      dirname(__FILE__);//获取当前文件的物理路径  
      dirname(__FILE__)."/../";//获取当前文件的上一级物理路径  
      ?>   
```
- 获取域名或主机地址

```
    echo $_SERVER['HTTP_HOST']."<br />";
```

- 获取网页地址

```
    echo $_SERVER['PHP_SELF']."<br />";
```

- 获取网址参数

```
    echo $_SERVER["QUERY_STRING"]."<br />";
```

- 获取用户代理

```
    echo $_SERVER['HTTP_REFERER']."<br />";
```

- 获取完整的url

```
    echo 'http://'.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'];
    echo 'http://'.$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF'].'?'.$_SERVER['QUERY_STRING'];
```

- 包含端口号的完整url
    
```
    echo 'http://'.$_SERVER['SERVER_NAME'].':'.$_SERVER["SERVER_PORT"].$_SERVER["REQUEST_URI"];
      
```

- 只取路径

```
    $url='http://'.$_SERVER['SERVER_NAME'].$_SERVER["REQUEST_URI"];
    echo dirname($url);
    
    
    
    echo 'SERVER_NAME：'.$_SERVER['SERVER_NAME'];  //获取当前域名（不含端口号）
    echo '<p>';
     
    echo 'HTTP_HOST：'.$_SERVER['HTTP_HOST'];//获取当前域名  （含端口号）
    echo '<p>';
      
    echo 'REQUEST_URI：'.$_SERVER['REQUEST_URI'];//获取当前域名的后缀 
```



# 日期相关

## 获取日期

### 获取本月日期：

```
    function getMonth($date){
      $firstday = date("Y-m-01",strtotime($date));
      $lastday  = date("Y-m-d",strtotime("$firstday +1 month -1 day"));
      
      return array($firstday,$lastday);
    }
```

$firstday是月份的第一天，假如$date是2014-2这样的话，$firstday就会是2014-02-01，然后根据$firstday加一个月就是2014-03-01，再减一天就是2014-02-28

### 获取上月日期：

```
    function getlastMonthDays($date){
      $timestamp=strtotime($date);
      $firstday =date('Y-m-01',strtotime(date('Y',$timestamp).'-'.(date('m',$timestamp)-1).'-01'));
      $lastday =date('Y-m-d',strtotime("$firstday +1 month -1 day"));
      return array($firstday,$lastday);
    }
```

上月日期需要先获取一个时间戳，然后在月份上-1就OK了，超智能的date()会把2014-0-1这种东西转换成2013-12-01


### 获取下月日期：

```
    function getNextMonthDays($date){
      $timestamp=strtotime($date);
      $arr=getdate($timestamp);
      if($arr['mon'] == 12){
        $year=$arr['year'] +1;
        $month=$arr['mon'] -11;
        $firstday=$year.'-0'.$month.'-01';
        $lastday=date('Y-m-d',strtotime("$firstday +1 month -1 day"));
      }else{
        $firstday=date('Y-m-01',strtotime(date('Y',$timestamp).'-'.(date('m',$timestamp)+1).'-01'));
        $lastday=date('Y-m-d',strtotime("$firstday +1 month -1 day"));
      }
      return array($firstday,$lastday);
    }
```

下月日期的代码看起来比较长一点，因为date()转不了类似2014-13-01这种东西，它会直接回到1970，所以前面需要处理一下12月的问题，除了12月就直接月份+1就OK啦


## 计算两个时间戳之间相差的时间

```
    //功能：计算两个时间戳之间相差的日时分秒
    //$begin_time  开始时间戳
    //$end_time 结束时间戳
    function timediff($begin_time,$end_time)
    {
          if($begin_time < $end_time){
             $starttime = $begin_time;
             $endtime = $end_time;
          }else{
             $starttime = $end_time;
             $endtime = $begin_time;
          }
    
          //计算天数
          $timediff = $endtime-$starttime;
          $days = intval($timediff/86400);
          
          //计算小时数
          $remain = $timediff%86400;
          $hours = intval($remain/3600);
          
          //计算分钟数
          $remain = $remain%3600;
          $mins = intval($remain/60);
          
          //计算秒数
          $secs = $remain%60;
          $res = array("day" => $days,"hour" => $hours,"min" => $mins,"sec" => $secs);
          return $res;
    }
    
    print_r(timediff(strtotime(2016-09-12 12:00:00'),strtotime('2016-09-15 21:50:21')));
```

## 判断是否 时间戳

```
    public static function isTimestamp($timestamp): bool
    {
        return (
            is_integer($timestamp)
            && ($timestamp >= 0)
            && ($timestamp <= 2147472000)
        );
    }
```

** :bool 是PHP7.1+ 的特性**

## 判断是否 同一天

```
    public function isSameDay($day1, $day2)
    {
        if (!$day1 || !$day2) {
            return false;
        }

        $day1 = is_numeric($day1) ? date('Y-m-d H:i:s', $day1) : $day1;
        $day2 = is_numeric($day2) ? date('Y-m-d H:i:s', $day2) : $day2;

        return (
            (new \Datetime($day1))->format('Y-m-d')
            ===
            (new \Datetime($day2))->format('Y-m-d')
        );
    }
```

# 距离/经纬度相关

## 计算某个经纬度的周围某段距离的正方形的四个点

```
    /**
     *	计算某个经纬度的周围某段距离的正方形的四个点
     *	地球半径，平均半径为6371km
     *	@param lng float 经度
     *	@param lat float 纬度
     *	@param distance float 该点所在圆的半径，该圆与此正方形内切，默认值为0.5千米
     *	@return array 正方形的四个点的经纬度坐标
     */
    
     function returnSquarePoint($lng, $lat,$distance = 0.5){
     
        $dlng =  2 * asin(sin($distance / (2 * 6371)) / cos(deg2rad($lat)));
        $dlng = rad2deg($dlng);
         
        $dlat = $distance/6371;
        $dlat = rad2deg($dlat);
         
        return array(
                    'left-top'=>array('lat'=>$lat + $dlat,'lng'=>$lng-$dlng),
                    'right-top'=>array('lat'=>$lat + $dlat, 'lng'=>$lng + $dlng),
                    'left-bottom'=>array('lat'=>$lat - $dlat, 'lng'=>$lng - $dlng),
                    'right-bottom'=>array('lat'=>$lat - $dlat, 'lng'=>$lng + $dlng)
                    );
     }
```

# 返回值相关

## JSON

```
    public static function jsonResp(
        $data,
        int $status = 200,
        bool $unicode = true
    ) {
        $unicode = $unicode ? JSON_UNESCAPED_UNICODE : null;

        $data = json_encode($data, $unicode);

        return response($data)
        ->header('Content-Type', 'application/json; charset=utf-8');
    }
```

## JSON Response

```
    public function jsonResponse($data)
    {
        if (! headers_sent()) {
            header('Content-Type: application/json; charset=UTF-8');
        }

        echo json_encode(
            $data,
            JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE
        );

        exit;
    }
```

# 数据格式相关

## xmlToArray

```
    public static function xmlToArray(string $xml)
    {
        return json_decode(json_encode(simplexml_load_string(
            $xml,
            'SimpleXMLElement',
            LIBXML_NOCDATA
        )), true);
    }
```

## array2XML

```
    public static function array2XML(array $array, string &$xml): string
    {
        foreach ($array as $key => &$val) {
            if (is_array($val)) {
                $_xml = '';
                $val = self::array2XML($val, $_xml);
            }
            $xml .= "<$key>$val</$key>";
        }

        unset($val);

        return $xml;
    }
```

## strtodate

```
    public static function strtodate($day)
    {
        if (! is_numeric($day)) {
            // Here we only need the `date` part only
            $day = strtotime(date('Y-m-d', strtotime($day)));
        }

        return date('Y-m-d H:i:s', strtotime(date('Y-m-d', $day)));
    }
```

# JWT 相关

## 生成 JWT

```
    public function getJWTString($params = [])
    {
        $header  = base64_encode(json_encode([
            'typ' => 'JWT',
            'alg' => 'SHA256',
        ]));
        $claims = [
            'exp' => __TIME+604800,    // 1 week
            'nbf' => __TIME,
            'iat' => __TIME,
        ];
        $payload = base64_encode(json_encode(array_merge($params, $claims)));
        $signature  = base64_encode(hash_hmac('sha256', $header.'.'.$payload, __CFG::JWT_SECRET_KEY));

        return implode('.', [$header, $payload, $signature]);
    }
```

## issue JWT

```
    public function issue($params = [])
        {
            $header  = base64_encode(json_encode([
                'typ' => 'JWT',
                'alg' => 'SHA256',
            ]));
            $timestamp = time();
            $claims = [
                'exp' => $timestamp+604800,    // One week
                'nbf' => $timestamp,
                'iat' => $timestamp,
            ];
            $payload    = base64_encode(json_encode(array_merge(
                $params,
                $claims
            )));
            $signature  = base64_encode(hash_hmac(
                'sha256',
                $header.'.'.$payload,
                $this->getSecureKeyOfOldSys()
            ));
    
            return implode('.', [$header, $payload, $signature]);
        }
```

## check JWT

```
    public function check($jwt)
        {
            $jwtComponents = explode('.', $jwt);
            if (3 != count($jwtComponents)) {
                return false;
            }
    
            list($header, $payload, $signature) = $jwtComponents;
            if ($headerArr = json_decode(base64_decode($header), true)) {
                if (is_array($headerArr) && isset($headerArr['alg'])) {
                    $alg = strtolower($headerArr['alg']);
                    if (in_array($alg, hash_algos())) {
                        if (base64_decode($signature) === hash_hmac(
                            $alg,
                            $header.'.'.$payload,
                            $this->getSecureKeyOfOldSys())
                        ) {
                            $data = json_decode(base64_decode($payload), true);
                            // Missing expire date or wrong JWT
                            if (! isset($data['exp'])
                                && !$this->isTimestamp($data['exp'])
                            ) {
                                return false;
                            }
                            // JWT expired
                            // !!! Make sure equal timezone were used both in JWT issuing and JWT checking
                            if (time() > $data['exp']) {
                                return false;
                            }
    
                            return $data;
                        }
                    }
                }
            }
    
            return false;
        }
```

# cURL相关

## HTTP

```
    public function http($url, $params=array(), $method='POST')
        {
            if(!function_exists('curl_init')){
                throw new NotFoundHttpException('请安装curl扩展');
            }
    
            $http = curl_init();
            /* Curl settings */
            curl_setopt($http, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_1_0);
            curl_setopt($http, CURLOPT_USERAGENT, $this->useragent);
            curl_setopt($http, CURLOPT_CONNECTTIMEOUT, $this->connect_timeout);
            curl_setopt($http, CURLOPT_TIMEOUT, $this->timeout);
            curl_setopt($http, CURLOPT_RETURNTRANSFER, TRUE);
            curl_setopt($http, CURLOPT_ENCODING, "");
            curl_setopt($http, CURLOPT_SSL_VERIFYPEER, FALSE);
            curl_setopt($http, CURLOPT_HEADER, FALSE);
    
            $params = http_build_query($params);
            switch ($method) {
                case 'POST':
                    curl_setopt($http, CURLOPT_POST, TRUE);
                    if (!empty($params)) {
                        curl_setopt($http, CURLOPT_POSTFIELDS, $params);
                    }
                    break;
                case 'PUT' :
                    curl_setopt($http, CURLOPT_PUT, true);
                    if (!empty($params)) {
                        $url = strpos('?',$url)===false ? "{$url}?{$params}" : "{$url}&{$params}";
                    }
                    break;
                case 'DELETE':
                    curl_setopt($http, CURLOPT_CUSTOMREQUEST, 'DELETE');
                    if (!empty($params)) {
                        $url = strpos('?',$url)===false ? "{$url}?{$params}" : "{$url}&{$params}";
                    }
                    break;
                case 'GET':
                    curl_setopt($http, CURLOPT_CUSTOMREQUEST, 'GET');
                    if (!empty($params)) {
                        $url = strpos('?',$url)===false ? "{$url}?{$params}" : "{$url}&{$params}";
                    }
            }
    
            $headers[] = "API-ClientIP: " . $_SERVER['REMOTE_ADDR'];
    
            curl_setopt($http, CURLOPT_URL, $url );
            curl_setopt($http, CURLOPT_HTTPHEADER, $headers );
            curl_setopt($http, CURLINFO_HEADER_OUT, TRUE );
            $res = curl_exec($http);
    
            // 检查是否有错误发生
            if(!curl_errno($http)) {
                $info = curl_getinfo($http);
            }
    
            curl_close($http);
            return $res;
        }
```

## 模拟 POST 请求

```
    public function postJsonApiByCurl($uri, $headers, $paramStr)
        {
            $ch = curl_init();
            curl_setopt_array($ch, [
                CURLOPT_URL        => $uri,
                CURLOPT_HTTPHEADER => $headers,
                CURLOPT_POST       => true,
                CURLOPT_POSTFIELDS => $paramStr,
                CURLOPT_RETURNTRANSFER => true,
            ]);
            $res = curl_exec($ch);
    
            $errNo  = curl_errno($ch);
            $errMsg = curl_error($ch);
    
            curl_close($ch);
    
            return [
                'errNo'  => $errNo,
                'errMsg' => $errMsg,
                'res'    => json_decode($res, true),
            ];
        }
```

或者:

```
    public function requestJsonApi(
            $uri,
            $type = 'POST',
            $params = [],
            $assoc = false
        )
        {
            $setOpt = [
                CURLOPT_URL        => $uri,
                CURLOPT_HTTPHEADER => [
                    'Content-Type: application/json; Charset=UTF-8',
                ],
                CURLOPT_RETURNTRANSFER => true,
                CURLOPT_USERAGENT => 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36',
                CURLOPT_AUTOREFERER => true,
                CURLOPT_FOLLOWLOCATION => true,
                CURLOPT_VERBOSE => true,
                CURLOPT_CONNECTTIMEOUT => 10,
                CURLOPT_TIMEOUT => 10,
                CURLOPT_SSL_VERIFYPEER => false,
                CURLOPT_SSL_VERIFYHOST => false,
            ];
    
            if ('POST' == $type) {
                $setOpt[CURLOPT_POST] = true;
                $setOpt[CURLOPT_POSTFIELDS] = $params;
            }
    
            $ch = curl_init();
            curl_setopt_array($ch, $setOpt);
            $res = curl_exec($ch);
            $errNo  = curl_errno($ch);
            $errMsg = curl_error($ch);
            curl_close($ch);
    
            return [
                'err' => $errNo,
                'msg' => $errMsg,
                'res' => json_decode($res, $assoc),
            ];
        }
```

## requestJsonApi

```
    public function requestJsonApi(
            $uri,
            $type = 'POST',
            $params = [],
            $headers = []
        ) {
            $headers = [
                'Content-Type: application/json; Charset=UTF-8',
            ];
    
            $res = $this->requestHTTPApi($uri, $type, $headers, $params);
    
            if (! $res['err']) {
                $res['res'] = json_decode($res['res'], true);
            }
    
            return $res;
        }
```

## requestHTTPApi

```
    public function requestHTTPApi(
            string $uri,
            string $type = 'GET',
            array $headers = [],
            $data
        ) {
            $setOpt = [
                CURLOPT_URL            => $uri,
                CURLOPT_RETURNTRANSFER => true,
            ];
    
            if ($headers) {
                $setOpt[CURLOPT_HTTPHEADER] = $headers;
            }
    
            if ('POST' == $type) {
                $setOpt[CURLOPT_POST]       = true;
                $setOpt[CURLOPT_POSTFIELDS] = $data;
            }
    
            $ch = curl_init();
            curl_setopt_array($ch, $setOpt);
            $res = curl_exec($ch);
    
            $errNo  = curl_errno($ch);
            $errMsg = curl_error($ch);
    
            curl_close($ch);
    
            return [
                'err' => $errNo,
                'msg' => ($errMsg ?: 'ok'),
                'res' => $res,
            ];
        }
```

# 数据验证相关

## 验证手机号码

```
    public function mobileZhFormatIsWrong($mobile)
    {
        // $ptn  = '/^13[0-9]{1}[0-9]{8}$|';
        // $ptn .= '14[1579]{1}[0-9]{8}$|';
        // $ptn .= '15[0-9]{1}[0-9]{8}$|';
        // $ptn .= '17[0135678]{1}[0-9]{8}$|';
        // $ptn .= '18[25789]{1}[0-9]{8}$/';
        return !preg_match('/^1[3-9]\d{9}$/u', $mobile);
    }
```

## 验证 HTTP_AUTHORIZATION

```
    // Authorise in HTTP HEADER `AUTHORIZATION`
    public function authorise()
    {
        if (! isset($_SERVER['HTTP_AUTHORIZATION'])
            || !is_string($_SERVER['HTTP_AUTHORIZATION'])
        ) {
            return false;
        }

        return $this->check($_SERVER['HTTP_AUTHORIZATION']);
    }
```

# 其他

## 生成交易号

```
    public static function tradeNo(
            $mid = 0,
            $mtype = '01',
            $domain = '00'
        ): string
        {
            $domain  = str_pad(($domain%42), 2, '0', STR_PAD_LEFT);
            $mid     = str_pad(($mid%1024), 4, '0', STR_PAD_LEFT);
            $mtype   = in_array($mtype, ['01', '02', '03']) ? $mtype : '00';
            $postfix = mb_substr(microtime(), 2, 6);
    
            return date('YmdHis').$domain.$mtype.$mid.mt_rand(1000, 9999).$postfix;
        }
```

## 计算两点骑行距离 

```
    public function getBicyclingDistanceInAmap($from, $to)
        {
            if (empty(env('AMAP_KEY'))) {
                throw new \Exception('Missing AMAP KEY');
            }
    
            $key = env('AMAP_KEY');
            $api = <<< EOF
    http://restapi.amap.com/v4/direction/bicycling?origin={$from}&destination={$to}&key={$key}
    EOF;
            $ret = $this->requestJsonApi($api, 'GET');
            if ($ret = (isset($ret['res']) ? $ret['res'] : false)) {
                if ($ret = (isset($ret['data']['paths'][0])
                    ? $ret['data']['paths'][0] : false
                )) {
                    if (isset($ret['distance']) && is_numeric($ret['distance'])) {
                        return $ret['distance'];
                    }
                }
            }
    
            return false;
        }
```

## 计算经纬度距离

```
    public function getDistance($lng1, $lat1, $lng2, $lat2)
        {
            //计算经纬度距离
            //将角度转为狐度
            $radLat1 = deg2rad($lat1);//deg2rad()函数将角度转换为弧度
            $radLat2 = deg2rad($lat2);
            $radLng1 = deg2rad($lng1);
            $radLng2 = deg2rad($lng2);
            $a = $radLat1 - $radLat2;
            $b = $radLng1 - $radLng2;
            $s = 2 * asin( sqrt ( pow (sin($a / 2),2) + cos($radLat1) * cos($radLat2) * pow(sin($b / 2),2) ) ) * 6378.137 * 1000;
            $s = round($s,2);
    
            return ($s < 1000) ? ( round($s, 2) . 'm') : round( intval($s / 1000).'.'.( $s % 1000), 2).'km';
        }
```

## 验证快钱支付

```
    // re-packing from kuaiqian demo for mobile
        public function kqPayCallbackSignVerify($params, $cert)
        {
            // by rmb_demo_php @kuaiqian
            function kq_ck_null($kq_va, $kq_na) {
                if ($kq_va == "") {
                    return $kq_va="";
                } else {
                    return $kq_va=$kq_na.'='.$kq_va.'&';
                }
            }
            //人民币网关账号，该账号为11位人民币网关商户编号+01,该值与提交时相同。
            $kq_check_all_para=kq_ck_null($params['merchantAcctId'], 'merchantAcctId');
            //网关版本，固定值：v2.0,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['version'], 'version');
            //语言种类，1代表中文显示，2代表英文显示。默认为1,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['language'], 'language');
            //签名类型,该值为4，代表PKI加密方式,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['signType'], 'signType');
            //支付方式，一般为00，代表所有的支付方式。如果是银行直连商户，该值为10,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['payType'], 'payType');
            //银行代码，如果payType为00，该值为空；如果payType为10,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['bankId'], 'bankId');
            //商户订单号，,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['orderId'], 'orderId');
            //订单提交时间，格式：yyyyMMddHHmmss，如：20071117020101,该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['orderTime'], 'orderTime');
            //订单金额，金额以“分”为单位，商户测试以1分测试即可，切勿以大金额测试,该值与支付时相同。
            $kq_check_all_para.=kq_ck_null($params['orderAmount'], 'orderAmount');
            $kq_check_all_para.=kq_ck_null($params['bindCard'], 'bindCard');
            $kq_check_all_para.=kq_ck_null($params['bindMobile'], 'bindMobile');
            // 快钱交易号，商户每一笔交易都会在快钱生成一个交易号。
            $kq_check_all_para.=kq_ck_null($params['dealId'], 'dealId');
            //银行交易号 ，快钱交易在银行支付时对应的交易号，如果不是通过银行卡支付，则为空
            $kq_check_all_para.=kq_ck_null($params['bankDealId'], 'bankDealId');
            //快钱交易时间，快钱对交易进行处理的时间,格式：yyyyMMddHHmmss，如：20071117020101
            $kq_check_all_para.=kq_ck_null($params['dealTime'], 'dealTime');
            //商户实际支付金额 以分为单位。比方10元，提交时金额应为1000。该金额代表商户快钱账户最终收到的金额。
            $kq_check_all_para.=kq_ck_null($params['payAmount'], 'payAmount');
            //费用，快钱收取商户的手续费，单位为分。
            $kq_check_all_para.=kq_ck_null($params['fee'], 'fee');
            //扩展字段1，该值与提交时相同
            $kq_check_all_para.=kq_ck_null($params['ext1'], 'ext1');
            //扩展字段2，该值与提交时相同。
            $kq_check_all_para.=kq_ck_null($params['ext2'], 'ext2');
            //处理结果， 10支付成功，11 支付失败，00订单申请成功，01 订单申请失败
            $kq_check_all_para.=kq_ck_null($params['payResult'], 'payResult');
            //错误代码 ，请参照《人民币网关接口文档》最后部分的详细解释。
            $kq_check_all_para.=kq_ck_null($params['errCode'], 'errCode');
    
            $trans_body = mb_substr($kq_check_all_para, 0, mb_strlen($kq_check_all_para)-1);
            $MAC = base64_decode($params['signMsg']);
    
    //        $fp = fopen("./99bill[1].cert.rsa.20140803.cer", "r");
    //        $cert = fread($fp, 8192);
    //        fclose($fp);
    
            $pubkeyid = openssl_get_publickey($cert);
    
            return openssl_verify($trans_body, $MAC, $pubkeyid);
        }
```

## 生成一个二维码图片

```
    public function generateQRCodeImage($content, $errorLevel = 'L', $pointSize = 10, $margin = 1)
    {
        if (!class_exists('QRcode')) {
            include_once __CORE_DIR.'libs/qrcode/phpqrcode.php';
        }

        QRcode::png($content, false, $errorLevel, $pointSize, $margin);

        exit;
    }
```

## PHP 定时运行脚本

```
    #!/usr/bin/env php
    
    <?php
    
    if ('cli' != php_sapi_name()) {
        exit('cli only');
    }
    @ini_set('display_errors', 'On');
    @date_default_timezone_set('Asia/Shanghai');
    set_time_limit(0);
    require(dirname(dirname(__FILE__)).'/system/home/index.php');
    $system = new Index('magic-shell');
    define('LOG_PATH', realpath(__DIR__.'/../').'/system/data/logs/');
    
    while (true) {
        autoRunFunc();  //需要运行的方法
        sleep(1);
    }
    
    // 记录日志
    function log_script($filename, $content) {
        $path = LOG_PATH.$filename.'.log';
        if (file_exists(LOG_PATH) && $filename && $content) {
            file_put_contents(
                $path,
                date('Y-m-d H:i:s')
                .' => '
                .$content
                .PHP_EOL,
                FILE_APPEND
            );
        }
    }
    
    function autoRunFunc()
    {
        // do sth
    }
```

# 参考

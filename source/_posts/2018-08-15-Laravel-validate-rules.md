---
title: Laravel 常见的验证规则
date: 2018-08-15 11:46:31
categories: [Laravel]
tags: [Laravel, validate]
---

> 记录几个常见的验证规则

<!-- more -->

>  我们将使用 `Illuminate\Http\Request` 对象提供的 `validate` 方法 。如果验证通过，你的代码就可以正常的运行。但是如果验证失败，就会抛出异常，并自动将对应的错误响应返回给用户。在典型的 HTTP 请求的情况下，会生成一个重定向响应，而对于 AJAX 请求则会发送 JSON 响应。

```
    use Illuminate\Http\Request;
    public function validate_rules(Request $request)
    {
        $this->validate($request, [
            'username' => 'required|string',  // 必须, 字符串
            'password' => 'required|digits_between:6,18',  // 验证的字段的长度必须在给定的 min 和 max 之间。
            'passwd'   => 'required|min:6|max:18',
            'mobile'   => 'required|numeric|regex:/^1[0-9]{10}$/',  // 手机号码正则验证
            'ip'       => 'required|ip',  // 验证的字段必须是 IP 地址。
            'avatar'   => 'required|dimensions:min_width=100,min_height=200', // 验证的文件必须是图片并且图片比例必须符合规则：
            'file'     => 'required|file', // 验证的字段必须是成功上传的文件。
            'images'   => 'required|image', // 验证的文件必须是一个图像（ jpeg、png、bmp、gif、或 svg ）
            'photo'    => 'required|mimes:jpeg,bmp,png', // 验证的文件必须具有与列出的其中一个扩展名相对应的 MIME 类型
            'title'    => 'bail|required|unique:posts|max:255',  // 在第一次验证失败后停止
            'email'    => 'required|email',  // 验证的字段必须符合 e-mail 地址格式。
            'email_db'    => 'required|connection.users,email_address', // 自定义数据库连接
            'start_date'  => 'required|date|after:tomorrow',  // 验证的字段必须等于给定日期之后的值
            'finish_date' => 'required|date|after:start_date',
            'is_accepted' => 'required|accepted',  // 验证的字段必须为 yes、 on、 1、或 true
            'verify_code' => 'required|alpha',  // 纯字母验证
            'is_array'    => 'required|array',  // 数组验证
            'is_bool'     => 'required|boolean',  // 验证的字段必须能够被转换为布尔值。可接受的参数为 true、false、1、0、"1" 以及 "0"
            'order_status' =>'required|in:1,2,3', // 验证的字段必须包含在给定的值列表中
        ]);

        // 验证成功,继续逻辑
    }
```

> 详细请看: https://laravel-china.org/docs/laravel/5.5/validation/1302#76a35d

# 别的验证写法

```
    protected function validate(array $params, array $rules)
    {
        $validator = Validator::make($params, $rules);

        if ($validator->fails()) {
            return [
                'err' => 400,
                'msg' => $validator->errors()->first(),
            ];
        }

        return true;
    }
```

```
    protected function verifyUserParams($params,$rules)
    {
        if (!is_array($params)
            || empty($params)
            || !is_array($rules)
            || empty($rules)) {
            return false;
        }

        $validator = Validator::make($params, $rules);

        if ($validator->fails()) {
            $this->_msg = $validator->errors()->first();
            return false;
        }

        return true;
    }

    protected $_msg = 'request not allow';
    
    $rules = [
        'id' => 'required'
    ];

    if (!$this->verifyUserParams($params,$rules) || $params['id'] <= 0) {
        return Tool::jsonResp([
            'err' => 2011,
            'msg' => '参数错误',
            'dat' => ''
        ]);
    }
```
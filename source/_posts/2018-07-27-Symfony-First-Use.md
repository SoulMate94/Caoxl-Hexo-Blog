---
title: Symfony 框架初次使用
date: 2018-07-27 16:51:14
categories: Symfony
tags: [Symfony]
---

> `Symfony` 框架的中文翻译是**交响乐**, 而`Composer`的中文翻译是**作曲家**, 这就让我非常有兴趣去学习一波
> [Symfony 中文文档](http://www.symfonychina.com/doc/current/setup.html)

<!-- more -->

# 安装 `Symfony Installer`

## `Linux`和`MAC OS X`系统

```
    sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
    
    sudo chmod a+x /usr/local/bin/symfony
```

## `Windows`系统

```
    c:\> php -r "readfile('http://symfony.com/installer');" > symfony
```

它会下载一个`symfony`文件，然后把这文件移动到你想创建`Symfony`项目的文件夹里，通过下述命令可引导各种安装

```
    c:\> move symfony c:\projects
    c:\projects\> php symfony
```

> 略略略, 直接进入`Composer`

## 用`Composer`创建`Symfony`程序

```
    composer create-project symfony/framework-standard-edition my_project_name
```

- 若需指定版本，提供版本号作为`create-project`的第二个参数

```
    composer create-project symfony/framework-standard-edition my_project_name "3.0.*"
```


# 运行`Symfony`

```
    php ./bin/console server:run
```

然后，打开浏览器访问`http://localhost:8000/`链接，即可看到`Symfony`欢迎页

# 创建一个页面

## 路由和控制器

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Routing\Annotation\Route;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\HttpFoundation\JsonResponse;
    
    class LuckyController
    {
        /**
         * @return Response
         * @Route("/lucky/number")
         */
        public function getRandNumber()
        {
            $num = rand(1, 100);
    
            return new Response(
                '<html><body>number: ' . $num . '</body></html>'
            );
        }
    }
```

> 访问 http://localhost:8000/lucky/number

## 路由在哪里 ?

> 路由写在注释里, 真的是长见识了, Symfony的路由居然是在注释里面
> > `@Route("/lucky/number")`
> > `@Route`被称为`annotation`，它定义了`URL`匹配


## 创建一个`JSON`响应

```
    <?php
    
    class LuckyController
    {
        /**
         * @return Response
         * @Route("/api/lucky/number2")
         */
        public function getRandNumberApi()
        {
            $data = array(
                'lucky_number' => rand(0, 100),
            );
    
            return new Response(
                json_encode($data),
                200,
                array('Content-Type' => 'application/json')
            );
        }
    }
```

> 访问: `http://localhost:8000/api/lucky/number2`

**你更可将代码精简为超好用的`JsonResponse`:**

```
    <?php
    
    // 使用JSONResponse需要引入下面这个声明
    use Symfony\Component\HttpFoundation\JsonResponse;
    
    class LuckyController
    {
        /**
         * @return Response
         * @Route("/api/lucky/number2")
         */
        public function getRandNumberApi()
        {
            $data = array(
                'lucky_number' => rand(0, 100),
            );
            
            return new JsonResponse($data);
        }
    }
```

## 动态URL匹配: `/lucky/number/{count}`

```
    /**
     * @param $count
     * @return Response
     * @Route("/lucky/number3/{count}")
     */
    public function getRandNumberCount($count)
    {
        $numbers = array();

        for ($i = 0; $i < $count; $i++) {
            $numbers[] = rand(0, 100);
        }

        $numbersList = implode(',', $numbers);

        return new Response(
            '<html><body>Lucky numbers: '.$numbersList.'</body></html>'
        );
    }
```


> 因为`{count}`占位符，页面`URL`变得不一样了。现在它要求`URLs`匹配`/lucky/number/*`

> 访问: `http://localhost:8000/lucky/number3/7`

## 渲染模板 (利用容器)

> 如果你在控制器中返回HTML，你可能需要渲染模板。幸运的是，**Symfony拥有Twig**

> 目前，`LuckyController`没有继承任何基类。此时引用`Twig`（或其他Symfony工具）最简单的方式，就是继承`Symfony`的`Controller`基类
  
- `Controller`

```
        /**
         * @param $count
         * @return Response
         * @Route("/lucky/number4/{count}")
         */
        public function getRandNumberTwig($count)
        {
            $numbers = array();
    
            for ($i = 0; $i < $count; $i++) {
                $numbers[] = rand(0, 100);
            }
    
            $numbersList = implode(',', $numbers);
    
            $html = $this->render(
                'lucky/number.html.twig',
                array('luckyNumberList' => $numbersList)
            );
    
            return new Response($html);
        }
```

- `Twig`

```
   {# app/Resources/views/lucky/number.html.twig #}
   {% extends'base.html.twig' %}
   
   {% block body %}
       <h1>Lucky Numbers: {{ luckyNumberList }}</h1>
   {% endblock %} 
```

> > 访问: `http://localhost:8000/lucky/number4/7`


# `Symfony` 目录结构

```
    ├─app
    │  ├─config
    │  └─Resources
    │      └─views
    │          ├─default
    ├─bin
    │  ├─console
    │  └─symfony_requirements
    ├─src
    │  └─AppBundle
    │      └─Controller
    ├─tests
    │  └─AppBundle
    │      └─Controller
    ├─var
    ├─vendor
    └─web
    
```

## `app/`

> 内含配置文件和模板. 大体上, 只要不是PHP代码的材料都放在这里

## `bin/`

> 用于存放二进制（binary）文件。最重要的是`console`文件，它被用来在`console`中执行`Symfony`命令。
> > php ./bin/console server:run 

## `src/`

> 你的PHP程序之所在

`src/`目录下暂时只有一个目录 - `src/AppBundle` - 所有的东西都在这里面

## `test/`

> 你程序的自动测试（如`Unit tests`/`单元测试`）被存放在这里。

## `var/`

> 这是那些自动生成的文件被存放的地方，比如缓存文件 (`var/cache/`) 和日志文件(`var/logs/`)

## `vendor/`

> 通过依赖管理器`Composer`，第三方类库、包、`bundles`被下载到这里。你应该不去编辑这个目录下的东西。

## `web/`

> 它是整个项目的文档根目录，存放可公开访问的文件，比如`CSS`、图片以及用来执行app（`app_dev.php`和`app.php`）的`Symfony`的前端控制器（`front controller`）。

# 路由

## 路由实例

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Routing\Annotation\Route;
    
    class BlogController extends Controller
    {
        /**
         * @Route("/blog", name="blog_list")
         */
        public function listAction()
        {
    
        }
    
        /**
         * @param $slug
         * @Route("/blog/{slug}", name="blog_show")
         */
        public function showAction($slug)
        {
    
        }
    }
```

## 添加{通配符}条件

> 设想 `blog_list` 路由将包含博客主题带有分页的一个列表，对于第2和第3页有类似 `/blog/2` 和 `/blog/3` 这样的URL。如果你把路由路径改为 `/blog/{page}`，会出现问题:

- `blog_list`: `/blog/{page}` 将匹配 `/blog/*`;
- `blog_show`: `/blog/{slug}` 将 同样 匹配 `/blog/*`。

**于是:**

> 要修复这个，添加一个 `requirement`（条件），以便 `{page}` 通配符能够 仅 匹配数字(digits):

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Routing\Annotation\Route;
    
    class BlogController extends Controller
    {
        /**
         * @Route("/blog", name="blog_list", requirements={"page": "\d+"})
         */
        public function listAction()
        {
    
        }
    
        /**
         * @param $slug
         * @Route("/blog/{slug}", name="blog_show")
         */
        public function showAction($slug)
        {
    
        }
    }
```

## 给{占位符}一个默认值

> 前例中，`blog_list` 路由的路径是 `/blog/{page}`。如果用户访问 `/blog/1`，则会匹配。但如果他们访问的是 `/blog`，就将 无法 匹配到。一旦你添加了某个 {占位符} 到路由中，它就 必须 得有一个值

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Routing\Annotation\Route;
    
    class BlogController extends Controller
    {
        /**
         * @Route("/blog", name="blog_list", requirements={"page": "\d+"})
         */
        public function listAction($page = 1)
        {
    
        }
    }
```

> 现在，当用户访问 `/blog` 时，`blog_list` 路由会匹配，并且 `$page` 路由参数会默认取值为 `1`。

### 高级路由示例

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Routing\Annotation\Route;
    
    class ArticleController extends Controller
    {
        /**
         * @param $_locale
         * @param $year
         * @param $slug
         * @Route(
         *     "/article/{_locale}/{year}/{slug}.{_format}",
         *     defaults={"_format": "html"},
         *     requirements={
         *          "_locale": "en|fr",
         *          "_format": "html|rss",
         *          "year": "\d+"
         *     }
         * )
         */
        public function showAction($_locale, $year, $slug)
        {
    
        }
    }
```

如你所见，这个路由仅在`URL`的 `{_locale}` 部分是 `en` 或 `fr` 时，同时 `{year}` 是一个数字的时候，才会匹配。此路由也表明，你可以在占位符之间使用一个“点”而不是一个斜杠。匹配这个路由的URL可能是下面这种：

- `/articles/en/2018/my-post`
- `/articles/fr/2018/my-post.rss`
- `/articles/en/2018/my-latest-post.html`

> !!! 一个路由占位符名称不可以起于数字，也不可以长于32个字符。

### 特殊路由参数

你已经看到，每一个路由参数或其默认值最终都是可以做为参数（`arguments`）用在控制器方法（译注：即`action`）中的。此外，还有四个特殊参数：每一个都能为你的程序增添一个独有的功能性。

- `_controller`: 就像你看到的, 此参数用于决定, 当路由匹配时, 须执行哪个控制器
- `_format`: 用于设置`request format`
- `_fragment`: 用于设置`fragment identifier`，即，`URL`中可选的最末部分，起自一个 # 字符，用来识别页面中的某一部分。
- `_locale`: 用于设置请求中的`locale`信息


## 控制器命名法

> bundle:controller:action

例如，`AppBundle:Blog:show` 这个 `_controller` 值代表:

- `Bundle (Bundle名)` - `AppBundle`
- `Controller Class (控制器类名)` - `BlogController`
- `Method Name (控制器方法名)` - `showAction`

```
    namespace AppBundle\Controller;
     
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
     
    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            // ...
        }
    }
```


## 加载路由

> `Symfony` 从一个 单一的 路由配置文件: `app/config/routing.yml`中加载你的程序中的全部路由
> 但从这个文件中，你可以加载任何一个 其他的 路由文件。实际上，`Symfony`默认从你 `AppBundle` 中的 `Controller/` 目录中加载`annotation`路由配置，这就是为何`Symfony`能够看到我们的`annotation`路由:

- `app\config\routing.yml`

```
    app:
        resource: '@AppBundle/Controller/'
        type: annotation
```

> `annotation`: 注释的意思

## 生成`URL`

路由系统也可用来生成`URL`链接。现实中，路由是个双向系统：**把URL映射到控制器，或把路由反解为URL**。

```
    /**
     * @param $slug
     * @Route("/blog/{slug}", name="blog_show")
     */
    public function showAction($slug)
    {
        $url = $this->generateUrl(
            'blog_show',
            array('slug' => 'my-blog-post')
        );
    }
```

- `generateUrl()`

```
    protected function generateUrl($route, $parameters = array(), $referenceType = UrlGeneratorInterface::ABSOLUTE_PATH)
    {
        return $this->container->get('router')->generate($route, $parameters, $referenceType);
    }
```

定义在 `Controller` 基类中的 `generateUrl()` 方法是以下代码的快捷方式:

```
    $url = $this->container->get('router')->generate(
        'blog_show',
        array('slug' => 'my-blog-post')
    );
```

### 生成带有`Query`字符串的`URL`

> `generate()` 方法接收通配符的值的数组，以生成URI。但如果你传入额外的值，它们将被添加到`URI`中，作为`query string`（查询字符串）:

```
    $this->get('router')->generate('blog', array(
        'page' => 2,
        'category' => 'Symfony'
    ));
```

### 生成绝对`URL`

> 默认时，路由生成相对链接 (如 `/blog`)。在控制器中把 `UrlGeneratorInterface::ABSOLUTE_URL` 作为 `generateUrl()` 方法的第三个参数传入:

```
    use Symfony\Component\Routing\Generator\UrlGeneratorInterface;
    
    $this->gengerateUrl('blog_show', array('slug' => 'my-blog-post'), UrlGeneratorInterface::ABSOLUTE_URL)
```

# 控制器

## 基础控制器

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Response;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    
    class TestController extends Controller
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }
```

## 重定向

```
    public function indexAction()
    {
        return $this->redirectToRoute('homepage');
        
        return $this->redirectToRoute('homepage', array(), 301);
    }
```

## 渲染模板

```
    public function indexAction()
    {
        return $this->render('default/index.html.twig', array('name' => $name));
    }
```

## 访问其他服务

当继承`controller`基类后，你可以通过`get()`方法访问任何`Symfony`的服务。下面列举了一些常见服务：

```
    $templating = $this->get('templating');
    
    $router = $this->get('router');
    
    $mailer = $this->get('mailer');
```

**问**: 到底存在哪些服务？我想要看所有的服务，请使用`debug:container`命令行查看：

```
    php bin/console debug:container
```


## `Request`对象作为一个控制器参数

```
    use Symfony\Component\HttpFoundation\Request;
    
    public function indexAction($firstName, $lastName, Request $request)
    {
        $page = $request->query->get('page', 1);
    }
```


## 管理`Session`

> `Symfony`提供了一个好用的`Session`对象，它能够存储有关用户的信息（它可以是使用浏览器的人、`bot`或`Web`服务）之间的请求。默认情况下，`Symfony`通过使用`PHP`的原生`Session`来保存`cookie`中的属性。

```
    <?php
    
    namespace AppBundle\Controller;
    
    use Symfony\Component\HttpFoundation\Request;
    
    class Session
    {
        public function indexAction(Request $request)
        {
            $session = $request->getSession();
    
            $session->set('foo', 'bar');
    
            $fooBar = $session->get('foobar');
    
            $filters = $session->get('filters', array());
        }
    }
```

## 请求和响应对象

正如前面所提到的，框架的`Request`对象会作为控制器的参数传入并强制指定数据类型为`Request`类：

```
    public function indexList(Request $request)
    {
        // is Ajax request ?
        $request->isXmlHttpRequest();

        $request->getPreferredLanguage('en', 'fr');

        $request->query->get('page');
        $request->request->get('page');

        $request->server->get('HTTP_HOST');

        $request->files->get('foo');

        $request->cookies->get('PHPSESSID');

        $request->headers->get('host');
        $request->headers->get('content_type');
    }
```


# 配置

> `Symfony`程序是由一组“负责呈现全部功能和可能性”的`bundles`所构成。每个`bundle`都可以通过`YAML`、`XML`或`PHP`格式的配置文件进行自定义

> 默认的主力配置文件是在`app/config`/目录下，它可以是`config.yml`、`config.xml`或`config.php`，根据你的偏好而定：

## 配置示例

### `YAML`

```
    # app/config/config.yml
    imports:
        - { resource: parameters.yml }
        - { resource: security.yml }
    
    framework:
        secret:          "%secret%"
        router:          { resource: "%kernel.root_dir%/config/routing.yml" }
        # ...
     
    # Twig Configuration
    twig:
        debug:            "%kernel.debug%"
        strict_variables: "%kernel.debug%"
     
    # ...
```

### `XML`

```
    <!-- app/config/config.xml -->
    <?xml version="1.0" encoding="UTF-8" ?>
    <container xmlns="http://symfony.com/schema/dic/services"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:framework="http://symfony.com/schema/dic/symfony"
        xmlns:twig="http://symfony.com/schema/dic/twig"
        xsi:schemaLocation="http://symfony.com/schema/dic/services
            http://symfony.com/schema/dic/services/services-1.0.xsd
            http://symfony.com/schema/dic/symfony
            http://symfony.com/schema/dic/symfony/symfony-1.0.xsd
            http://symfony.com/schema/dic/twig
            http://symfony.com/schema/dic/twig/twig-1.0.xsd">
     
        <imports>
            <import resource="parameters.yml" />
            <import resource="security.yml" />
        </imports>
     
        <framework:config secret="%secret%">
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework:config>
     
        <!-- Twig Configuration -->
        <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%" />
     
        <!-- ... -->
    </container>
```

### `PHP`

```
    // app/config/config.php
    $this->import('parameters.yml');
    $this->import('security.yml');
     
    $container->loadFromExtension('framework', array(
        'secret' => '%secret%',
        'router' => array(
            'resource' => '%kernel.root_dir%/config/routing.php',
        ),
        // ...
    ));
     
    // Twig Configuration
    $container->loadFromExtension('twig', array(
        'debug'            => '%kernel.debug%',
        'strict_variables' => '%kernel.debug%',
    ));
     
    // ...
```

## 环境配置

- `AppKernel`类负责加载你指定的配置文件：

```
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(function (ContainerBuilder $container) {
            $container->setParameter('container.autowiring.strict_mode', true);
            $container->setParameter('container.dumper.inline_class_loader', true);

            $container->addObjectResource($this);
        });
        $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
    }
```
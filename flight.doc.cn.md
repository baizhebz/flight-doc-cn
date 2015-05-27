# Flight是什么？

Flight是一个快速，简易，可扩展的PHP框架。Flight能使你快速和轻松地创建RESTful Web应用。

```php
require 'flight/Flight.php';

Flight::route('/', function(){
    echo 'hello world!';
});

Flight::start();
```


# 需求

Flight需要`PHP 5.3`或更高版本。

# License

Flight is released under the [MIT](http://flightphp.com/license) license.

# 安装

1\.框架下载
如果你在使用[Composer](https://getcomposer.org/)，你可以运行如下命令：

```
composer require mikecao/flight
```

或者你可以直接[下载](https://github.com/mikecao/flight/tarball/master)，之后将Flight框架文件放入你的web目录中。

2\. 配置你的web服务器

对于*Apache*服务器，编辑你的`.htaccess`文件添加如下内容：

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

对于*Nginx*服务器，添加如下内容到你的server声明中：

```
server {
    location / {
        try_files $uri $uri/ /index.php;
    }
}
```
3\. 创建你的`index.php`文件（示例）

首先引入这个框架。

```php
require 'flight/Flight.php';
```

接着定义一个路由并且注册一个函数去处理对应请求。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

最后，启动框架。

```php
Flight::start();
```

# 路由

Flight中的路由是指将一个URL模式(pattern)匹配到一个回调函数中。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

只要能被调用，都可以当做回调函数。所以可以使用一个普通的函数当做回调：

```php
function hello(){
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

也可以是某一个类的方法：

```php
class Greeting {
    public static function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', array('Greeting','hello'));
```

如果定义了多个路由，路由依照定义它们的顺序进行匹配。第一个匹配到该请求的路由将被调用。

## HTTP METHOD路由

在默认不指定的情况下，路由会对相应请求的所有Method(例如：GET POST PUT DELETE...)进行匹配。你可以通过在URL前面加一个方法标识符的方式来响应指定的Method。

```php
Flight::route('GET /', function(){
    echo 'I received a GET request.';
});

Flight::route('POST /', function(){
    echo 'I received a POST request.';
});
```

你还可以使用`|`分隔符来映射多个Method到同一个回调中。

```php
Flight::route('GET|POST /', function(){
    echo 'I received either a GET or a POST request.';
});
```

## 正则表达式

在路由中你可以使用正则表达式：

```php
Flight::route('/user/[0-9]+', function(){
    // 这个将匹配到 /user/1234
});
```

## 命名参数

你可以在路由中指定命名参数，它们会被传递到你的回调函数里。

```php
Flight::route('/@name/@id', function($name, $id){
    echo "hello, $name ($id)!";
});
```

你也可以通过使用`:`分隔符在命名变量中引入正则表达式

```php
Flight::route('/@name/@id:[0-9]{3}', function($name, $id){
    // 这个将匹配到 /bob/123
    // 但是不会匹配到 /bob/12345
});
```

## 可选参数

你可以通过将URL段(segments)包在括号里的方式来指定哪些命名参数是可选的。

```php
Flight::route('/blog(/@year(/@month(/@day)))', function($year, $month, $day){
    // 它将匹配如下URLS:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
});
```

任何没有被匹配到的可选参数将以NULL值传入。

## 通配符

匹配只发生在单独的URL段(segments)。如果你想匹配多段，可以使用`*`通配符。

```php
Flight::route('/blog/*', function(){
    // 这个将匹配到 /blog/2000/02/01
});
```

要将所有的请求路由到单一的回调上，你可以这么做：

```php
Flight::route('*', function(){
    // Do something
});
```

## 路由的传递

当从一个被匹配到的回调函数中返回`true`时，路由功能将继续执行，传递到下一个能匹配的路由中。

```php
Flight::route('/user/@name', function($name){
    // 检查某些条件
    if ($name != "Bob") {
        // 延续到下一个路由
        return true;
    }
});

Flight::route('/user/*', function(){
    // 这里会被调用到
});
```

## 路由信息

如果你想检视匹配到的路由信息，可以请求将路由对象传递到你的回调函数中：你需要把
route方法的第三个参数设置成`true`。这个路由对象总是会作为最后一个参数传入你的回调函数。

```php
Flight::route('/', function($route){
    // 匹配到的HTTP方法的数组
    $route->methods;

    // 命名参数数组
    $route->params;

    // 匹配的正则表达式
    $route->regex;

    // Contains the contents of any '*' used in the URL pattern
    $route->splat;
}, true);
```

# 扩展

Fligth被设计成一个可扩展的框架。这个框架带来了一系列的默认方法和组件，但是它允许你
映射你自己的方法，注册你自己的类，甚至可以重写已有的类和方法。

## 方法的映射

你可以使用`map`函数去映射你自定义的方法：

```php
// 映射你自己的方法
Flight::map('hello', function($name){
    echo "hello $name!";
});

// 调用你的自定义方法
Flight::hello('Bob');
```

## 类的注册

你可以使用`register`函数去注册你自己的类：

```php
// 注册你定义的类
Flight::register('user', 'User');

// 得到你定义的类的一个实例
$user = Flight::user();
```

register方法允许你向类的构造函数传递参数。所以当你加载自定义类的时候，它将会
预初始化(pre-initialized)。你可以通过一个追加的数组来传递定义的构造函数参数。
这是一个加载数据库连接的例子：

```php
// 注册一个带有构造函数参数的类
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'));

// 得到你定义的类的一个实例
// 这里将创建一个带有你定义的参数的对象
//
//     new PDO('mysql:host=localhost;dbname=test','user','pass');
//
$db = Flight::db();
```

如果你传递了额外的回调函数参数，它将会在类构造完之后立即执行。这就允许你为这个新对象去
执行任何的安装过程(set up procedures)。这个回调函数会被传递一个参数，就是这个新对象的实例。

```php
// 这个回调函数将会传递到这个被构造的对象中
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'), function($db){
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
});
```

默认情况下，每次你加载一个类，你会得到一个共享的实例。如果要得到一个类的新实例，
简单的传递一个`false`参数就行了。

```php
// 类的共享实例
$shared = Flight::db();

// 类的新实例
$new = Flight::db(false);
```

需要记住的是，被映射的方法优先于被注册的类。如果你用相同的名字将它们都声明了，那么只有
映射的方法会被调用。

# 重写(Overriding)

Flight允许你按照自己的需要去重写它的默认功能，而不用修改任何框架的代码。

例如，当Flight的路由功能没有匹配到一个URL时，它会调用`notFound`方法，发出一个通用的
`HTTP 404`响应。你可以使用`map`方法去重写这个行为。

```php
Flight::map('notFound', function(){
    // 显示自定义的404页面
    include 'errors/404.html';
});
```

Flight也允许你替换这个框架的核心组件。例如你可以将默认的Router类替换成你自定义的类：

```php
// 注册成你自定义的类
Flight::register('router', 'MyRouter');

// When Flight loads the Router instance, it will load your class
// 当Flight加载Router实例时，将会加载到你自定义的类
$myrouter = Flight::router();
```

然而框架的方法诸如`map`和`register`是不能够被重写的。如果你尝试这么做的话你会得到一个error。

# 过滤

Flight允许你在方法调用之前和之后去过滤它。框架里没有任何你需要记住的预定义的钩子。你可以
过滤任何被映射的自定义方法和框架中的方法。

一个过滤器函数是像这样的：

```php
function(&$params, &$output) {
    // Filter code
}
```

通过传入的变量，你可以操作输入参数和/或输出参数。

这样做就可以在一个方法运行之前运行一个过滤器：

```php
Flight::before('start', function(&$params, &$output){
    // Do something
});
```

这样做就可以在一个方法运行之后运行一个过滤器：

```php
Flight::after('start', function(&$params, &$output){
    // Do something
});
```

你可以给任何函数添加任意数量的过滤器。它们将按照声明的顺序依次被调用。

这里是一个过滤器处理的例子：

```php
// 映射一个自定义的方法
Flight::map('hello', function($name){
    return "Hello, $name!";
});

// 添加一个前置的过滤器
Flight::before('hello', function(&$params, &$output){
    // 操作这里的params
    $params[0] = 'Fred';
});

// 添加一个后置的过滤器
Flight::after('hello', function(&$params, &$output){
    // 操作这里的output
    $output .= " Have a nice day!";
});

// 调用这个自定义方法
echo Flight::hello('Bob');
```

这个将会输出：

    Hello Fred! Have a nice day!

如果你定义了多个过滤器，你可以通过在任意一个过滤器函数中返回`false`来终结这个过滤器链。

```php
Flight::before('start', function(&$params, &$output){
    echo 'one';
});

Flight::before('start', function(&$params, &$output){
    echo 'two';

    // 如下将会终止这个过滤器链
    return false;
});

// 这里将不会得到调用
Flight::before('start', function(&$params, &$output){
    echo 'three';
});
```

记住，核心函数诸如`map`和`register`是不能够被过滤的，因为它们是被直接调用而非动态调用的。

# 变量

Flight允许你定义变量，使得它能在应用内的任何地方被使用。

```php
// 保存你定义的变量
Flight::set('id', 123);

// 在应用的其他地方使用
$id = Flight::get('id');
```
去检测一个变量是否被设置了可以这么做：

```php
if (Flight::has('id')) {
     // Do something
}
```

去清除一个变量你可以这么做：

```php
// 清除这个id变量
Flight::clear('id');

// 清除所有的变量
Flight::clear();
```

Flight框架使用变量的目的还包括了配置。

```php
Flight::set('flight.log_errors', true);
```

# 视图

Flight默认提供了一些基础的模板功能。调用带有模板文件和
可选的模板数据的`render`函数，去显示一个视图模板。

```php
Flight::render('hello.php', array('name' => 'Bob'));
```

你传进来的模板数据，会被自动的注入到模板当中，并且可以像一个本地变量一样去引用。
模板文件就是简单的PHP文件。如果一个文件名为`hello.php`的模板文件的内容是这样的：

```php
Hello, '<?php echo $name; ?>'!
```

输出会是：

    Hello, Bob!

你可以使用set函数来手动的设置视图中的变量：

```php
Flight::view()->set('name', 'Bob');
```

这个`name` 变量现在在你所有的视图中都是可用的了。所以就可以简化成这样了：

```php
Flight::render('hello');
```

注意当你在render函数中指定模板名时，你可以去掉这个`.php`的扩展名。

默认情况下Flight会在`views`目录下寻找模板文件。你可以通过如下配置的设定来为你的模板
设置另外一个路径。

```php
Flight::set('flight.views.path', '/path/to/views');
```

## 布局(Layouts)

对网站来说，拥有一个单独的可交换内容的布局(layout)模板文件是很常见的。要在布局中使用渲染的内容，
你可以给`render`函数传递一个可选的参数。

```php
Flight::render('header', array('heading' => 'Hello'), 'header_content');
Flight::render('body', array('body' => 'World'), 'body_content');
```

紧接着你的视图就有了命名为`header_content`和`body_content`的已保存的变量。接下来你就可以
这样渲染你的布局了：

```php
Flight::render('layout', array('title' => 'Home Page'));
```

如果你的模板文件是这样的：

`header.php`:

```php
<h1><?php echo $heading; ?></h1>
```

`body.php`:

```php
<div><?php echo $body; ?></div>
```

`layout.php`:

```php
<html>
<head>
<title><?php echo $title; ?></title>
</head>
<body>
<?php echo $header_content; ?>
<?php echo $body_content; ?>
</body>
</html>
```

输出会是：
```html
<html>
<head>
<title>Home Page</title>
</head>
<body>
<h1>Hello</h1>
<div>World</div>
</body>
</html>
```

## 自定义视图

Flight允许你替换默认的视图引擎，只需简单的注册你自己的视图类即可。这里展示的是在视图中
如何使用[Smarty](http://www.smarty.net/)模板引擎：

```php
// 加载Smarty类库
require './Smarty/libs/Smarty.class.php';

// 将Smarty注册成视图类
// 同时传递一个回调函数，在加载过程中配置Smarty
Flight::register('view', 'Smarty', array(), function($smarty){
    $smarty->template_dir = './templates/';
    $smarty->compile_dir = './templates_c/';
    $smarty->config_dir = './config/';
    $smarty->cache_dir = './cache/';
});

// 模板中数据的赋值
Flight::view()->assign('name', 'Bob');

// 显示这个模板
Flight::view()->display('hello.tpl');
```

出于完备性，你还应该重写Flight的默认render方法：

```php
Flight::map('render', function($template, $data){
    Flight::view()->assign($data);
    Flight::view()->display($template);
});
```
# 错误(Error)处理

## 错误(Errors)和异常(Exceptions)

所有的errors和exceptions都会被Flight捕获，然后传到`error`方法。该方法默认的行为是
发出一个带有错误信息的通用的`HTTP 500 Internal Server Error`响应。

出于你自己的需要，你可以重写这个行为：

```php
Flight::map('error', function(Exception $ex){
    // 错误处理
    echo $ex->getTraceAsString();
});
```

默认情况下，错误(errors)是不会被记录日志到web服务器的。你可以通过改变配置来允许记录。

```php
Flight::set('flight.log_errors', true);
```

## Not Found

当一个URL没有被找到时，Flight将会调用`notFound`方法。该方法默认的行为是
发出一个通用的`HTTP 404 Not Found`响应并带有简单的说明信息。

出于你自己的需要，你可以重写这个行为：

```php
Flight::map('notFound', function(){
    // 处理not found
});
```

# 重定向(Redirects)

你可以使用`redirect`方法将当前请求重定向到传入的新URL中。

```php
Flight::redirect('/new/location');
```

默认情况下Flight会发出一个HTTP 303状态码。你可以选择设置一个自定义的状态码。

```php
Flight::redirect('/new/location', 401);
```

# 请求

Flight将HTTP请求封装到一个单独的对象中，你可以这样获取到它：

```php
$request = Flight::request();
```

request对象提供了如下的属性：

```
url - 被请求的url
base - The parent subdirectory of the URL
method - 请求的Method (GET, POST, PUT, DELETE)
referrer - 引用（referrer）的 URL
ip - 客户点的IP地址
ajax - 是否是一个ajax请求
scheme - 服务器scheme (http, https)
user_agent - 浏览器信息
type - Content-type
length - Content-length
query - 查询字符串参数（Query string parameters）
data - Post数据或者JSON数据
cookies - Cookies数据
files - 上传的文件
secure - Whether the connection is secure
accept - HTTP accept parameters
proxy_ip - 客户端代理ip地址
```

你可以通过数组或对象的方式来获取`query`,`data`,`cookies`和 `files`属性。

也就是说，你可以这样获取到查询字符串参数(query string parameter)：

```php
$id = Flight::request()->query['id'];
```

或者你可以这样做：

```php
$id = Flight::request()->query->id;
```

## 请求体原始数据(RAW Request Body)

要获取原始的HTTP请求体数据，举例来说当你正在处理PUT方法的请求时，你可以这么做：

```php
$body = Flight::request()->getBody();
```

## JSON 输入

如果你发送`application/json`类型的请求并带有数据`{"id": 123}`时，它将被从`data`属性中获取到。

```php
$id = Flight::request()->data->id;
```

# HTTP缓存

Flight为HTTP级别的缓存提供了内建的支持。如果满足缓存的条件，Flight将会返回一个
HTTP`304 Not Modified`响应。当下一次客户端请求相同的资源时，它们会被提示去使用它们
本地的缓存版本。

## Last-Modified

你可以使用`lastModified`方法并传递一个UNIX时间戳去设置一个页面最后被修改的日期和时间。
客户端将继续使用它们的缓存直到last modified的值被改变了。

```php
Flight::route('/news', function(){
    Flight::lastModified(1234567890);
    echo 'This content will be cached.';
});
```

## ETag

`ETag`缓存与`Last-Modified`类似，但你可以对资源指定任意的id。

```php
Flight::route('/news', function(){
    Flight::etag('my-unique-id');
    echo 'This content will be cached.';
});
```

需要记住的是，不论调用了`lastModified`或是`etag`，都会设置并且检查缓存的值。如果缓存中的值
跟请求的相同，Flight会立即发送一个`HTTP 304`响应并且停止处理。

# 停止

你可以通过调用`halt`方法在任何地方停止这个框架：

```php
Flight::halt();
```

你也可以指定可选的`HTTP`状态码和信息：

```php
Flight::halt(200, 'Be right back...');
```

调用`halt`将会丢弃在那个点之前的任何的响应内容。如果你想停止这个框架并输出当前的响应，使用`stop`方法：

```php
Flight::stop();
```

# JSON

Flight对发送JSON和JSONP响应提供了支持。发送一个JSON响应时，你传递的数据将被JSON编码。

```php
Flight::json(array('id' => 123));
```

对于JSONP请求，你可以选择传递查询参数名(query parameter name)用于定义你的回调函数：

```php
Flight::jsonp(array('id' => 123), 'q');
```

所以，当使用`?q=my_func`构造一个GET请求时，你应该会收到这样的输出：

```
my_func({"id":123});
```

如果你没有传递查询参数名(query parameter name)的话，它会有一个默认名`jsonp`。

# 配置

你可以使用`set`方法去设置配置的值，来自定义Flight的某些行为。

```php
Flight::set('flight.log_errors', true);
```

下面是所有的可进行设置的配置列表：

    flight.base_url - 覆盖该请求的base url。(默认值：null)
    flight.handle_errors - 允许Flight处理所有的内部错误。 (默认值：true)
    flight.log_errors - 向web服务器的错误日志文件里记录错误日志。 (默认值：false)
    flight.views.path - 包含视图模板文件的目录路径。 (默认值：./views)

# 框架的方法

Flight框架被设计成易于使用和易于理解的。下面就是这个框架完整的方法集合。它由 是常规静态函数
的核心方法，和被映射的可以被过滤和重写的扩展方法组成。

## 核心方法

```
Flight::map($name, $callback) // 创建一个自定的框架方法
Flight::register($name, $class, [$params], [$callback]) // 将一个类注册成框架方法Flight::before($name, $callback) // 添加框架方法的前置过滤器
Flight::after($name, $callback) // 添加框架方法的后置过滤器
Flight::path($path) // 添加类自动加载(autoloading)的路径
Flight::get($key) // 获取某个变量的值
Flight::set($key, $value) // 设置变量的值
Flight::has($key) // 某个变量是否被设值
Flight::clear([$key]) // 清除一个变量
Flight::init() // 初始化框架到默认的设定
Flight::app() // 获取整个应用对象的实例
```

## 扩展方法

```
Flight::start() // 开启框架（接收响应开始工作）
Flight::stop() // 框架停止并且发送返回响应
Flight::halt([$code], [$message]) // 停止框架并返回一个可选的http状态码和信息
Flight::route($pattern, $callback) // 将一个URL匹配到一个回调中
Flight::redirect($url, [$code]) // 重定向到另一个URL
Flight::render($file, [$data], [$key]) // 渲染模板文件
Flight::error($exception) // 发送HTTP 500响应
Flight::notFound() // 发送HTTP 404响应
Flight::etag($id, [$type]) // 运行HTTP Etag缓存
Flight::lastModified($time) // 运行HTTP last modified缓存
Flight::json($data, [$code], [$encode]) // 发送JSON响应
Flight::jsonp($data, [$param], [$code], [$encode]) // 发送JSONP响应
```

任何通过`map`和`register`添加的自定义方法都可以被过滤。

# 框架的实例

替代将Flight运行成一个全局的静态类，你可以选择将它运行成一个对象的实例。

```php
require 'flight/autoload.php';

use flight\Engine;

$app = new Engine();

$app->route('/', function(){
    echo 'hello world!';
});

$app->start();
```

也就是取代调用静态方法，你可以调用Engine对象实例里同名的方法。
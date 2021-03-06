---
layout: chapter
title:  应用程序
---
<div class="back"><a href="/tlboc/">&laquo; 返回目录</a></div>

#创建应用程序

既然你基本已经了解了CoffeeScript的语法，那现在让我们来探索下如何使用它来真枪实弹的创建应用程序。本节内容对所有的CoffeeScript开发者都有用，无论你是新人还是老手，实际上，这对只写JavaScript的人来说也非常有用。

不知什么原因，当开发者构建JavaScript客户端程序时，往往不会使用可靠的测试过的模式或约定，这就导致大量的代码像意大利面条一样无法维护。程序架构的重要性再怎么强调都不过分。如果你所写的JavaScript或者CoffeeScript不是表单验证这么简单的话，你需要使用某种程序架构，比方说[MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)。

创建可维护的大型应用程序的秘诀就是不要做大型程序。换句话说，就是创建一系列低耦和的模块。程序逻辑越通用越好，尽量抽象出来。最终，将你的逻辑分别拆分为视图（views）、数据模型（models）和控制器（controllers）（MVC）。如何实现MVC超出了本章的范围，需要的话我建议你到[JavaScript Web Applications](http://oreilly.com/catalog/9781449307530/)翻看我的书，或者使用像[Backbone](http://documentcloud.github.com/backbone/) 或者 [Spine](https://github.com/maccman/spine)这类框架。现在先不管那些，在这里我们使用CommonJS模块来构建应用程序。

## 结构 & CommonJS

那什么CommonJS模块到底是什么呢？好的，如果你用过[NodeJS](http://nodejs.org/) 的话，你大概没有意识到你在使用CommonJS模块。CommonJS模块起初是为了编写服务器端JavaScript类库而开发的，企图用它来解决模块加载、命名空间和作用域的问题。还有一个通用的形式用来兼容所有的JavaScript实现。目标是写一个给[Rhino](http://www.mozilla.org/rhino/) 的类库也可以用于Node。终于这种思想被移植到了浏览器中，而且现在我们有像[RequireJS](http://requirejs.org)和[Yabble](https://github.com/jbrantly/yabble)这样好的类库来构建模块化的客户端。

实际上，模块能保证你的代码运行在局部作用域中（代码分装），你可以通过`require()`函数来载入模块，而且通过`module.exports`来暴露模块。让我们更加深入的研究下。

###导入文件

你可是使用`require()`来载入其他模块和类库。只给它传递一个模块名即可，而且如果该模块正加载目录中，`require()`会返回一个代表该模块的对象。例如：

{% highlight coffeescript %}
    User = require("models/user")
{% endhighlight %}
    
同步的`require`是一个颇具争议的问题，但是主流的类库加载器和CommonJS的预案中已经解决了这个问题。如果你不想使用我下面推荐的Stitch而想另走一条路的话，你可能需要看一下它。

###暴露属性

默认情况下，模块不会暴露任何属性，因此模块内的东西对于`require()`调用来说不可见。如果你想访问模块的某个属性，你需要将它挂到`module.exports`：

{% highlight coffeescript %}
    # random_module.js
    module.exports.myFineProperty = ->
      # Some shizzle
{% endhighlight %}
    
现在，`require`这个模块的时候`myFineProperty`就会暴露出来：

{% highlight coffeescript %}
    myFineProperty = require("random_module").myFineProperty
{% endhighlight %}

##使用Stitch打包

把你的代码格式化为CommnonJS模块很不错，但是如何让这些模块在客户端也能工作呢？我采用[Stitch](https://github.com/sstephenson/stitch)这个不太有名的类库作为解决方案。Stitch的作者是Sam Stephenson，其思想来自于[Prototype.js](http://www.prototypejs.org)，非常优雅的解决了模块的问题，真让我兴奋呀！Stitch简单将所有的JavaScript文件打包到一起，巧妙的将它们包裹在CommonJS中，而不是尝试动态处理依赖。噢，我差点忘记说 ，它还能编译CoffeeScript、JS模板、[LESS CSS](http://lesscss.org)和[Sass](http://sass-lang.com)。 

首先，如果你必须安装 [Node.js](http://nodejs.org/)和[npm](http://npmjs.org/)，如果还没有安装的话。在本章中我们要用到它们。
    
现在，创建我们的程序结构。如果你正在使用[Spine](https://github.com/maccman/spine)，那你可以使用[Spine.App](http://github.com/maccman/spine.app)自动生成，否则你需要手动的创建。我通常把全部的程序代码放到`app`目录下，`lib`存放通常的类库，然后包括其他一些像静态资源等等放到`public`目录中。

{% highlight bash %}
    app
    app/controllers
    app/views
    app/models
    app/lib
    lib
    public
    public/index.html
{% endhighlight %}

接着，为了启动我们的Stitch服务，让我们创建一个名为`index.coffee`的文件，添加如下脚本：

<span class="csscript"></span>

{% highlight coffeescript %}
    require("coffee-script")
    stitch  = require("stitch")
    express = require("express")
    argv    = process.argv.slice(2)
    
    package = stitch.createPackage(
      # Specify the paths you want Stitch to automatically bundle up
      paths: [ __dirname + "/app" ]
      
      # Specify your base libraries
      dependencies: [
        # __dirname + '/lib/jquery.js'
      ]
    )
    app = express.createServer()
    
    app.configure ->
      app.set "views", __dirname + "/views"
      app.use app.router
      app.use express.static(__dirname + "/public")
      app.get "/application.js", package.createServer()

    port = argv[0] or process.env.PORT or 9294
    console.log "Starting server on port: #{port}"
    app.listen port
{% endhighlight %}
    
你会发现我们依赖了一些类库：`coffee-scirpt`、`stitch`和`express`。我们需要创建一个`package.json`文件，列出这些依赖类库以便npm可以将它们打包到一起。我们的`./package.json`看起来像下面这样：

{% highlight javascript %}
    {
      "name": "app",
      "version": "0.0.1",
      "dependencies": { 
        "coffee-script": "~1.1.2",
        "stitch": "~0.3.2",
        "express": "~2.5.0",
        "eco": "1.1.0-rc-1"
      }
    }
{% endhighlight %}
    
然后让我使用npm安装这些依赖：

{% highlight bash %}
    npm install .
    npm install -g coffee-script
{% endhighlight %}
    
好，我们就要完成了，现在运行：

{% highlight bash %}
    coffee index.coffee
{% endhighlight %}
    
但愿你的stitch服务器已经运行起来了，让我继续在`app`目录中添加一个`app.coffee`脚本来测试下。这个就是将要引导启动我们程序的文件。

<span class="csscript"></span>

{% highlight coffeescript %}
    module.exports = App =
      init: ->
        # Bootstrap the app
{% endhighlight %}
        
现在让我们创建主页`index.html`，如果我们创建的是单页面程序，那它将是唯一一个我们会访问的页面。这是一个静态资源，因此将其放在`public`目录下。

{% highlight html %}
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset=utf-8>
      <title>Application</title>
      <!-- Require the main Stitch file -->
      <script src="/application.js" type="text/javascript" charset="utf-8"></script>
      <script type="text/javascript" charset="utf-8">
        document.addEventListener("DOMContentLoaded", function(){
          var App = require("app");
          App.init();
        }, false);
      </script>
    </head>
    <body>
    </body>
    </html>
{% endhighlight %}

当页面加载完成，我们的*DOMContentLoaded*事件回调函数会require`app.coffee`脚本（它已经被自动编译好了），然后调用`init()`函数。其实就是这样，我们获取了一个CommonJS模块并且运行了它，就如一个HTTP服务器和CoffeeScript编译器一样。假如，我们想包含一个模块，只需调用`require()`即可。让我们创建一个新的类，`User`，在`app.coffee`中引用它：

<span class="csscript"></span>

{% highlight coffeescript %}
    # app/models/user.coffee
    module.exports = class User
      constructor: (@name) ->
      
    # app/app.coffee
    User = require("models/user")
{% endhighlight %}

##JavaScript模板

如果你把逻辑都放到了客户端，那你就必须使用某种模板引擎。JavaScript模板除了它运行在客户端之外,与服务器端的模板引擎非常相似，比方说Ruby的ERB或者Python的文本插值。有很多模板类库，因此我鼓励你进行研究将它们找出来。默认地，Stitch预置了对[Eco](https://github.com/sstephenson/eco)的支持。

JavaScript模板与服务端模板非常相似。你可以混合使用模板标签和HTML，在模板绘制过程中这些标签会被求值被替换。[Eco](https://github.com/sstephenson/eco)模板最好的地方在于，它们可以直接使用CoffeeScript。

例如：

{% highlight coffeescript %}
    <% if @projects.length: %>
      <% for project in @projects: %>
        <a href="<%= project.url %>"><%= project.name %></a>
        <p><%= project.description %></p>
      <% end %>
    <% else: %>
      No projects
    <% end %>
{% endhighlight %}

如你所见，模板语法非常明了。只需使用`<%`标签来执行表达式，`<%=`标签输出表达式的值。下面是一个部分模板标签的列表：
    
* `<% expression %>`  
  对一个CoffeeScript表达式求值，但不输出其返回值

* `<%= expression %>`  
  对一个CoffeeScript表达式求值，转义返回值，并将其输出。

* `<%- expression %>`  
  对一个CoffeeScript表达式求值，不转义，直接输出返回值。

你可以在模板标签中使用任意的CoffeeScript表达式，但是有一点需要注意，就是虽然CoffeeScript对空格敏感，但是Eco模板不敏感。因此，Eco模板标签在开始一个CoffeeScirpt缩进块时必须使用一个冒号作为后缀。然后使用一个特殊的标签`<% end %>`来表示缩进块结束。例如：

{% highlight coffeescript %}
    <% if @project.isOnHold(): %>
      On Hold
    <% end %>
{% endhighlight %}
    
`if`和`end`并非要写成多行：

{% highlight coffeescript %}
    <% if @project.isOnHold(): %> On Hold <% end %>
{% endhighlight %}

而且你也可以使用`if`单行后缀表达式来实现：

{% highlight coffeescript %}
    <%= "On Hold" if @project.isOnHold() %>
{% endhighlight %}

现在我们已经学习了语法，让我们在`views/users/show.eco`定义一个Eco模板吧：

{% highlight coffeescript %}
    <label>Name: <%= @name %></label>
{% endhighlight %}
    
Stitch会自动编译我们的模板且把它们打包到`application.js`中。然后，在程序控制器中我们就可以require模板，就像使用一个模块一样，传入所以的数据执行它。 
    
<span class="csscript"></span>

{% highlight coffeescript %}
    require("views/users/show")(new User("Brian"))
{% endhighlight %}
    
我们的`app.coffee`文件现在看起来像下面这样，当文档加载完毕后，我们渲染这个模板并将其添加到页面中：

<span class="csscript"></span>

{% highlight coffeescript %}
    User = require("models/user")

    App =
      init: ->
        template = require("views/users/show")
        view     = template(new User("Brian"))

        # Obviously this could be spruced up by jQuery
        element = document.createElement("div")
        element.innerHTML = view
        document.body.appendChild(element)
    
    module.exports = App
{% endhighlight %}
    
访问[http://localhost:9294/](http://localhost:9294/)，然后随便到处转转！希望这个教程能够教会你如何构建一个客户端的CoffeeScript程序。下一步，我推荐研究一些客户端模块，比方说[Backbone](http://documentcloud.github.com/backbone/)或者[Spine](http://spinejs.com)，它们能够为你提供一个基础的MVC框架，将你解放出来，做你感兴趣的事情。
    
##附加-使用Heroku 30秒快速发布

[Heroku](http://heroku.com/)是一个非常棒的Web主机服务提供商。它为你管理所有的服务器和扩展，让你能够做自己感兴趣的事情（创建有意思的JavaScript程序）。要让本教程的实例工作你需要一个Heroku账户，好消息是它基本套餐是免费的。之前Heroku提供的是Ruby主机服务，不过现在最近它发布了它的Cedar栈，支持Node。

首先，我们需要创建一个`Profile`文件，用它来向Heroku提供我们的程序信息。

{% highlight bash %}
    echo "web: coffee index.coffee" > Procfile
{% endhighlight %}

然后，你需要为你的程序创建一个本地的git代码仓库，如果你还没有创建的话。

{% highlight bash %}
    git init
    git add .
    git commit -m "First commit"
{% endhighlight %}    
    
现在发布程序，我们将使用`heroku`这个gem（如果你还没安装的话，你需要先安装好）。

{% highlight bash %}
    heroku create myAppName --stack cedar
    git push heroku master
    heroku open
{% endhighlight %}
    
好了，这样做就行了。没有比这还简单的Node主机的服务了。

##其他类库

[Stitch](https://github.com/sstephenson/stitch)和[Eco](https://github.com/sstephenson/eco)并不是唯一你可用于创建CoffeeScript和Node程序的类库，还有很多可作为替代的类库。

例如，当需要使用模板时，你可以使用 [Mustache](http://mustache.github.com)、[Jade](http://jade-lang.com)或者[CoffeeKup](http://coffeekup.org)（使用纯CoffeeScript写HTML）。

说到为程序提供服务，[Hem](http://github.com/maccman/hem)是比较好的选择，支持CommonJS和NPM模块，而且还无缝的集成了CoffeeScript MVC框架[Spine](http://spinejs.com)。[node-browsify](https://github.com/substack/node-browserify)是另外一个类似的项目。如果你想使用[express](http://expressjs.com/)集成的更底层的东西，Trevor Burnham的[connect-assets](https://github.com/TrevorBurnham/connect-assets)不错。

你可以在[项目的wiki](https://github.com/jashkenas/coffee-script/wiki/Web-framework-plugins)上找到一个CoffeeScript Web框架插件的完整列表。

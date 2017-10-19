# 1.安装
Epoxy依赖 jQuery 1.7.0或更高版本，Underscore 1.4.3或更高版本, 以及Backbone 0.9.9或更高版本. 下载Epoxy库（压缩版9k,gzip只有2k），把引入它的标签放到所有依赖的后面。

```javascript
<script src="js/jquery-min.js"></script>
<script src="js/underscore-min.js"></script>
<script src="js/backbone-min.js"></script>
<script src="js/backbone.epoxy.min.js"></script>
```
你可以选择用Zepto代替 jQuery，用Lo-Dash 代替 Underscore. 当你要兼容IE6/7时，记住引入json2.Epoxy 是遵循MIT许可的开源代码，你可以在Github Repo 中浏览库的源码。

# 2.简单的视图绑定
让我们开始做一个简单的绑定，当他们潜在的模型数据发生变化时，我们就改变这些DOM元素的内容.
```javascript
var bindModel = new Backbone.Model({
  firstName: "Luke",
  lastName: "Skywalker"
});

var BindingView = Backbone.Epoxy.View.extend({
  el: "#app-luke",
  bindings: {
    "input.first-name": "value:firstName,events:['keyup']",
    "input.last-name": "value:lastName,events:['keyup']",
    "span.first-name": "text:firstName",
    "span.last-name": "text:lastName"
  }
});

var view = new BindingView({model: bindModel});

```

```html
<div id="app-luke" class="demo">
  <label>First:</label>
  <input type="text" class="first-name">

  <label>Last:</label>
  <input type="text" class="last-name">

  <b>Full Name:</b>
  <span class="first-name"></span>
  <span class="last-name"></span>
</div>
```

结果是：

 在这个例子中，我们创建了一个新的 Epoxy.View实例，给它提供一个原生的 Backbone 模型，然后用绑定散列去声明模型属性和视图选择器之间的绑定。
绑定声明的格式是：“handler:dataSource”,一般这是一个键值对，键定义一个处理机来处理绑定，值是一个数据源来充当绑定。Epoxy 提供了一系列的处理器，你也可以自己添加。在最常见的绑定实现中，数据源引用视图模型上的属性。
在上面的例子中，value:firstName 建立了一个在input 值属性 和绑定的模型的firstName 属性之间的双向的绑定。同样地，text:firstName 建立了一个单向绑定，绑定的是模型属性到 绑定绑定元素的文本。最后，events:['keyup'] 用作声明一个DOM事件，这个事件使绑定在 除“change”事件以外做出响应。

##行内的绑定声明
另一个流行的数据绑定的方法是在目标DOM元素上的属性中声明绑定。这种方法把声明绑定的方法从视图中转移的DOM中，Epoxy也支持这种语法，你喜欢用那种就用那种。
**demo**

```javascript
var bindModel = new Backbone.Model({
  firstName: "Han",
  lastName: "Solo"
});

var BindingView = Backbone.Epoxy.View.extend({
  el: "#app-han",
  bindings: "data-bind"
});

var view = new BindingView({model: bindModel});
```
**html**
```html
<div id="app-han" class="demo">
  <label>First:</label>
  <input type="text" data-bind="value:firstName,events:['keyup']">

  <label>Last:</label>
  <input type="text" data-bind="value:lastName,events:['keyup']">

  <b>Full Name:</b>
  <span data-bind="text:firstName"></span>
  <span data-bind="text:lastName"></span>
</div>
```

 
在这里，我们强制用元素的属性来实现同样的绑定，视图中的绑定属性简单的定义查询用的属性名（注："data-bind" 是 Epoxy的默认的选择器，所以你不需要在你的视图中去特别的声明）。
以上两个例子在功能上是相同的，因此你可以选择你喜欢的声明方式，但建议把他们都声明在一个地方。这本指导的例子将使用行内绑定的形式来避免元素绑定的不清楚。当然，不要把这个解读成一个倾向，实际上在视图中声明绑定有很多的优势，比如保持所以的功能定义都在试图中，保持DOM的干净）。
想了解更多关于在视图中设置绑定，请参考**Epoxy.View 文档**

# 3.计算模型属性

# 4

# 5.计算的getter和setter
目前我们已经看到了一个用计算属性用做只读属性的 get 方法。现在我们写一个可读写的计算塑性，它即可以得到计算的值，也可以设置一个或多个值到模型中。
**html**
```html
<div id="app-readwrite">
  <label>Price (updates on blur):</label>
  <input type="text" data-bind="value:displayPrice">

  <b>Display Price:</b>
  <span data-bind="text:displayPrice"></span>

  <b>Model Price:</b>
  <span data-bind="text:price"></span>
</div>

```
**javascript**
```javascript
var PriceModel = Backbone.Epoxy.Model.extend({
  defaults: {
    productName: "Light Saber",
    price: 5000
  },
  computeds: {
    displayPrice: {
        get: function() {
            return "$"+this.get("price");
        },
        set: function( value ) {
            return {price: parseInt(value.replace("$", "")||0, 10)};
        }
    }
  }
});

var view = new Backbone.Epoxy.View({
  el: "#app-readwrite",
  model: new PriceModel()
});
```
这里，我们用带有get和set方法的参数对象定义我们的计算属性。get方法可以得到我们组装的模型值，set方法将改变输入的原始值然后同步到格式化的模型数据中。在上面的例子中，这个displayPrice 计算属性用它的get方法格式化货币字符串，然后再重新格式化输入的数据，用它的set方法在数据保存到模型之前把输入的值转化成一个正确的数字。
注意setter 方程返回一个属性散列而不是直接在它的模型中直接调用set方法。计算setter返回的属性通过set操作合并到模型中。这允许一个计算的setter定义多个属性的修改。这些属性和其他队列模型的改变是同步的。
了解更多的计算属性getter和setter，请参考 Model.addComputed 部分的文档。

# 6.计算的视图属性
当然计算属性能很好的管理数据，当数据需要格式化成特别的显示目的，计算属性就会出错；举个例子：当数据为了呈现需要格式化成html。这个例子对视图很特别，因此应该在视图中计算。
为了适应视图中的特殊格式化，一个Epoxy 视图可以定义它自己的能在在绑定中获得的一系列的计算属性。让我们来用视图计算属性来格式化一个值用作显示：
```html
<div id="app-view-computed">
  <label>First:</label>
  <input type="text" data-bind="value:firstName,events:['keyup']">

  <label>Last:</label>
  <input type="text" data-bind="value:lastName,events:['keyup']">

  <span data-bind="html:nameDisplay"></span>
</div>
```
```javascript
var ComputedView = Backbone.Epoxy.View.extend({
  el: "#app-view-computed",
  computeds: {
    nameDisplay: function() {
      return "<b>"+ this.getBinding("lastName") +"</b>, "+ this.getBinding("firstName");
    }
  }
});

var view = new ComputedView({
  model: new Backbone.Model({firstName: "Mace", lastName: "Windu"})
});
```
在上面的例子中计算的视图特性应该看起来很熟悉，视图计算器和模式和模型计算器类似，他们的不同在于数据的获取。对于视图计算器，我们用视图的getBinding 方法来获取数据。注意：我们在管理计算器依赖时讨论过的条件逻辑的警告依然适合视图计算器。
了解更多关于管理视图计算器属性和他们的依赖，请参考 **computed view properties**部分的文档

# 7 视图绑定过滤
Epoxy  视图在强绑定设置和清除绑定定义之间达到平衡。Epoxy和 Knockout.js 运用同样的技术，它不鼓励Knockout's 的行内javascript代码。
相反，Epoxy 为在绑定中直接格式化数据提供包装器。注意在下面的案例中 not（）和 format() 过滤器是怎么使用的。

```html
<div id="app-filters">
  <p>
  <label>First name*</label>
  <input type="text" data-bind="value:firstName,events:['keyup']">
  <span data-bind="toggle:not(firstName)">Please enter a first name.</span>
  </p>
  <p>
  <label>Last name*</label>
  <input type="text" data-bind="value:lastName,events:['keyup']">
  <span data-bind="toggle:not(lastName)">Please enter a last name.</span>
  </p>
  <p data-bind="text:format('Name: $1 $2',firstName,lastName)"></p>
  <p class="req">* Required</p>
</div>
```
```javascript
var view = new Backbone.Epoxy.View({
el: "#app-filters",
  model: new Backbone.Model({
    firstName: "Luke",
    lastName: "Skywalker"
  })
});
```
在上面的例子中，为了特殊的呈现包装器用作格式化绑定数据。not（）过滤器 用作否定一个值得真实性，而format（）过滤器用作合并多个值组成一个字符串通过一个熟悉的正则反向引用格式。
绑定过滤器唯一要注意的是不要嵌套。这个是Epoxy故意限制的，Epoxy的观点，应用逻辑不属于你绑定的声明。假如你需要一个经过多次处理的值，你应该用计算器属性提前处理，或用一个自定义的指令。
查看全部可用的绑定过滤器，或学习怎么定义自己的过滤器。

# 8.自定义指令


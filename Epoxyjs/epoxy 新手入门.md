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
绑定处理程序把数据的值应用到DOM中，Epoxy提供一系类的默认绑定处理器来应对很多基本的视图操作。对于其他内容，鼓励开发者为视图中特别的操作自己写绑定处理器。自定义绑定处理器很容易去定义：
```html
<div id="app-custom">
  <label>Millennium Falcon:</label>
  <input type="checkbox" value="Millennium Falcon" data-bind="checked:shipsList">

  <label>Death Star:</label>
  <input type="checkbox" value="Death Star" data-bind="checked:shipsList">
  <b>Ships:</b>
  <span data-bind="listing:shipsList"></span>
</div>
```
```javascript
var model = new Backbone.Model({shipsList: []});

var BindingView = Backbone.Epoxy.View.extend({
  el: "#app-custom",
  bindingHandlers: {
    listing: function( $element, value ) {
        $element.text( value.join(", ") );
    }
  }
});

var view = new BindingView({model: model});
```
在上面的例子中，我们设置一个叫做listing 自定义的绑定处理器来灵巧地打印一个数组的值。这个自定义的处理器将用在视图绑定的声明上，就是看到的  "listing:shipsList"绑定。
一个绑定处理器仅仅是一个接受两个参数的方法，第一个参数是绑定元素的jQuery对象，第二个元素是绑定到视图中的数据值。在一个自定义绑定处理器中，你仅仅声明一个方法，通过这个方法格式化数据，绑定值到元素中。注意，上面的例子仅仅是一个简单只读绑定。
在 View.bindingHandlers 文档中你可以了解更多关于 自定义处理器及 怎么配置一个双向的绑定。

# 9.绑定集合及多种资源
到目前为止，我们讨论了用模型属性绑定视图属性的基本运用实例。现在让我们探索添加额外的数据源，包括 Backbone.Collection实例。
首先，什么是数据源？一个数据源提供 自己和他属性的绑定内容，绑定内容是视图中的可得到的编译过的所有数据的列表。源数据既可以是 Backbone.Model实例也可以是Backbone.Collection 实例。默认情况，一个 Epoxy 视图 的模型和集合属性会默认配置一个数据源（假如你需要，你也可以添加额外的数据源）。
 
数据源以别名 "$sourceName" 包含在绑定上下文中。因此，视图模型和集合特性在绑定中以 $model and $collection被引用。这些直接的数据源引用用作集合绑定子在例子中。
```html
<div id="bind-collection">
  <ul data-bind="collection:$collection"></ul>
</div>
```
```javascript
var ListItemView = Backbone.View.extend({
  tagName: "li",
  initialize: function() {
    this.$el.text( this.model.get("label") );
  }
});

var ListCollection = Backbone.Collection.extend({
  model: Backbone.Model
});

var ListView = Backbone.Epoxy.View.extend({
  el: "#bind-collection",
  itemView: ListItemView,

  initialize: function() {
    this.collection = new ListCollection();
    this.collection.reset([{label: "Luke Skywalker"}, {label: "Han Solo"}]);
  }
});

var view = new ListView();
```
在上面的例子中，data-bind="collection:$collection" 绑定一个无序列表的内容到视图的数据源上. 然而用什么渲染集合的个体项？注意，在列表视图类中，子视图属性是怎么定义的。子视图属性为集合定义了一个 子项的渲染。
关于集合绑定的等多细节，请阅读集合处理器的文档。更多关于运用数据源绑定的例子，看下面的 Epoxy ToDos 例子。


# 10.Epoxy ToDos
每一个现在的 JavaScript MV* 框架，让我们用 Epoxy 视图绑定和原生的Backbone模型建立一个 小的 ToDos 应用

代码见：`http://epoxyjs.org/tutorials.html#epoxy-todos`
在应用中有4个组，包括：
**TodoItemModel ** ：这是一个用作存储每个todo数据的原生的 Backbone模型，在这个例子中，每个todo条目有一个标题和一个完成状态。
**TodoItemView**: 这个Epoxy 视图用作显示每一个todo条目列表。这个视图由一个checkbox和 和 text input 构成一个DOM片段，让后绑定元素的值到视图模型中。另外，视图添加了一些自定义的绑定处理器来管理视图，readonly 处理器触发 文本输入框的readonly属性，而 save :在元素的值发生变化后绑定调用它来保存到绑定的模型中。
`TodosCollection` : 这是一个用来管理我们活动的todo列表的原生Backbone collection。它的成员是 `TodoItemModel ` 模型构造出来的。 
`TodoAppView`：最后，这个Epoxy 视图管理主应用的容器视图。它用原生的Backbone 事件设置应用的原始控制，用作从 todos 集合实例中增加和删除子项。它也应用了一个Epoxy的集合，用它绑定视图的默认集合数据源（以 $collection引用）。也应注意：todo子视图为渲染每个集合向提供 子视图属性。

记住：这个应用没有要求只有使用了数据绑定才能工作。实际上，数据绑定对于常见的应用方案是过火的。当评估你的项目的目标和宗旨时记住。讽刺的是这个库的作者非常保守的推荐这个库，当数据绑定在你需要的时候是一个好的工具，但他不应该是一个你在创建界面时的自然选择；尤其是在使用Backbone时。当一个情景需要数据绑定...你会喜欢它做的一切。



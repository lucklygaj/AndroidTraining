# Data Binding Library 数据绑定库

* This document explains how to use the Data Binding Library to write declarative layouts and minimize the glue code necessary to bind your application logic and layouts
* 本文介绍了如何使用数据绑定库去写声明布局文件和减少绑定你的应用程序的逻辑和布局所必需的粘合代码。

* The Data Binding Library offers both flexibility and broad compatibility — it's a support library, so you can use it with all Android platform versions back to Android 2.1 (API level 7+).
* 数据绑定库是一个支持库，它提供了灵活性和广阔的兼容性，所以你可以在版本Android2.1之后的所有android平台中使用它。

* To use data binding, Android Plugin for Gradle 1.5.0-alpha1 or higher is required. [See how to update the Android Plugin for Gradle.](https://developer.android.com/studio/releases/gradle-plugin.html#updating-plugin)
* 使用数据绑定要求Android中的Gradle插件在1.5.0-alpha1 或者更高的版本。

# Build Environment 构建环境

* To get started with Data Binding, download the library from the Support repository in the Android SDK manager.
* 开始使用数据绑定库前先要在Android SDK 管理者中从支持库中下载该库。

* To configure your app to use data binding, add the dataBinding element to your build.gradle file in the app module.
* 在你的应用程序的模块中的build.gradle文件中添加dataBinding元素去配置你的应用程序去使用数据绑定库的功能。

* Use the following code snippet to configure data binding:
* 使用下面的代码片段去配置使用数据绑定
```
android {
    ....
    dataBinding {
        enabled = true
    }
}
```
* If you have an app module that depends on a library which uses data binding, your app module must configure data binding in its build.gradle file as well.
* 如果你有一个应用程序模块依赖了一个使用数据绑定的库，你的应用程序模块同样需要在build.gradle文件中配置数据绑定。

* Also, make sure you are using a compatible version of Android Studio. Android Studio 1.3 and later provides support for data binding as described in Android Studio Support for Data Binding.
* 此外，请确保你使用了一个兼容版本的Android Studio。Android Studio 1.3 之后的都为数据绑定提供了支持。

# Data Binding Layout Files 数据绑定布局文件

### Writing your first set of data binding expressions

### 写你的第一组数据绑定表达式 
* Data-binding layout files are slightly different and start with a root tag of layout followed by a data element and a view root element. This view element is what your root would be in a non-binding layout file. A sample file looks like this:
* 数据绑定布局文件略有不同，在布局开始的根标签后要跟着一个数据元素和一个根视图元素。你的根视图元素jiang将在一个不具约束力的不惧文件中。看起来就想下面的示例文件：
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```
* The user variable within data describes a property that may be used within this layout.
* 使用数据变量描述一个可以在这个布局文件内使用的属性。

```
<variable name="user" type="com.example.User"/>
```
* Expressions within the layout are written in the attribute properties using the "@{}" syntax. Here, the TextView's text is set to the firstName property of user:
* 在布局文件内使用"@{}"语法写属性，在这里，TextView的text设置为user的firstName属性。

```
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>
```
# Data Object 数据对象

* Let's assume for now that you have a plain-old Java object (POJO) for User:
* 让我们假设现在你有一个简单的User Java对象（POJO）

```
public class User {
   public final String firstName;
   public final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
}
```

* This type of object has data that never changes. It is common in applications to have data that is read once and never changes thereafter. It is also possible to use a JavaBeans objects:
* 这种类型的对象有一个永远不会改变的数据。这种数据在应用程序中是常见的，这种数据一旦呗读取一次就永远不会被改变。此外也可以使用JavaBeans对象。

```
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

* From the perspective of data binding, these two classes are equivalent. The expression @{user.firstName} used for the TextView's android:text attribute will access the firstName field in the former class and the getFirstName() method in the latter class. Alternatively, it will also be resolved to firstName() if that method exists.
* 从数据绑定的观点来看，这两个类是等价的。TextView的android：text属性通过使用表达式@{user.firsName}将获得模版类中的firsName字段和后面的类中的getFirsName()方法。或者，它也可以被解析为firstName()，如果该方法存在。

# Binding Data 绑定数据

* By default, a Binding class will be generated based on the name of the layout file, converting it to Pascal case and suffixing "Binding" to it. The above layout file was main_activity.xml so the generate class was MainActivityBinding. This class holds all the bindings from the layout properties (e.g. the user variable) to the layout's Views and knows how to assign values for the binding expressions.The easiest means for creating the bindings is to do it while inflating:

* 默认的，将会根据布局文件的名字生成一个绑定类，将其转换为Pascal的实例并在名字后面添加Binding作为后缀.上述的布局文件是main_activity.xml,所以生成类叫做MainActivityBinding.该类控制来自布局的视图中的布局属性中的所有绑定并且知道如何去为绑定表达式分配值。在渲染布局时创建绑定是最简单的方式。

```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```
* You're done! Run the application and you'll see Test User in the UI. Alternatively, you can get the view via:
* 你已经完成！运行应用程序你将在UI中看到测试的用户信息。或者，你可以通过视图其获得。

``` 
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());
```
* If you are using data binding items inside a ListView or RecyclerView adapter, you may prefer to use:

* 如果你正在使用数据绑定一个ListView或者RecyclerView适配器中的项目，你可能更喜欢使用：

```
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
//or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

```

# Event Handling 事件处理

* Data Binding allows you to write expressions handling events that are dispatched from the views (e.g. onClick). Event attribute names are governed by the name of the listener method with a few exceptions. For example, View.OnLongClickListener has a method onLongClick(), so the attribute for this event is android:onLongClick. There are two ways to handle an event.

 * 数据绑定允许你写表达式去处理来自视图（点击）的转发的事件。通过监听方法宁子管理事件属性名单，有些例外。例如，View.onLongClickListener有一个onLongCkick()方法，所以对于该事件的属性是android:onLongClick.以下是两个方式去处理事件。

1. Method References: In your expressions, you can reference methods that conform to the signature of the listener method. When an expression evaluates to a method reference, Data Binding wraps the method reference and owner object in a listener, and sets that listener on the target view. If the expression evaluates to null, Data Binding does not create a listener and sets a null listener instead.
 * 方法引用：在你的表达式中，你可以引用符合监听方法签名的方法。当一个表达式评估一个方法引用时，数据绑定将方法的引用和监听器中拥有的对象包裹起来并且将监听器设置给目标对象。如果表达式评估是Null，数据绑定不会创建一个监听器而是设置一个值为null的监听器合取代

2. Listener Bindings: These are lambda expressions that are evaluated when the event happens. Data Binding always creates a listener, which it sets on the view. When the event is dispatched, the listener evaluates the lambda expression.

* 监听器绑定:当事件发生时，lambda表达式将被评估。数据绑定总是会创建一个监听者，该监听者会被设置到view上。当事件转发的时候，监听者评估该kambda表达式。

## Method References 方法引用
* Events can be bound to handler methods directly, similar to the way android: onClick can be assigned to a method in an Activity. One major advantage compared to the View#onClick attribute is that the expression is processed at compile time, so if the method does not exist or its signature is not correct, you receive a compile time error.
* 事件可以被直接绑定到处理方法上，与在一个Actuvity中使用android：onClick可以被分配到一个方法上的方式类似。与设置View#onClick属性相比一个主要的优势是该表达式在编译时被处理，如果该方法不存在或者它的签名不正确，你将收到一个编译时错误。

* The major difference between Method References and Listener Bindings is that the actual listener implementation is created when the data is bound, not when the event is triggered. If you prefer to evaluate the expression when the event happens, you should use listener binding.
* 方法的引用和监听者的绑定这两种方式主要的不同是当数据绑定时被监听者的是被真实的创建，而不是在事件触发时。如果你更倾向与在事件发生时去评估该表达式，你应该使用监听者绑定。

* To assign an event to its handler, use a normal binding expression, with the value being the method name to call. For example, if your data object has two methods:

* 使用值是作为方法名的正常的绑定表达式去分配一个事件给处理程序。例如，如果你的数据对象有两个方法。
```
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```
* The binding expression can assign the click listener for a View:

* 绑定表达式可以分配给视图的点击监听者:
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```
* Note that the signature of the method in the expression must exactly match the signature of the method in the Listener object.
* 注意在表达式中方法的签名必须准确匹配监听者对象中方法的签名

# Listener Bindings 监听者绑定
* Listener Bindings are binding expressions that run when an event happens. They are similar to method references, but they let you run arbitrary data binding expressions. This feature is available with Android Gradle Plugin for Gradle version 2.0 and later.
* 监听者的绑定是在一个事件发生时运行绑定的绑定表达式。他们类似方法的引用，但它们允许你运行任意的数据绑定表达式，该特性在使用Android Gradle插件且Gradle版本为2.0之后是可用的。

* In method references, the parameters of the method must match the parameters of the event listener. In Listener Bindings, only your return value must match the expected return value of the listener (unless it is expecting void). For example, you can have a presenter class that has the following method:
* 在方法的绑定中，方法的参数必须匹配事件监听者的参数。在监听者绑定中，只有你的返回值必须匹配监听者预期返回的值（除非它的愈切返回值是空）。例如，你可以有一个持久化的类，该类有以下方法：
```
public class Presenter {
    public void onSaveClick(Task task){}
}
```

* Then you can bind the click event to your class as follows:
* 然后你可以使用如下代码绑定点时间到你的类。

```
  <?xml version="1.0" encoding="utf-8"?>
  <layout xmlns:android="http://schemas.android.com/apk/res/android">
      <data>
          <variable name="task" type="com.android.example.Task" />
          <variable name="presenter" type="com.android.example.Presenter" />
      </data>
      <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
          <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
          android:onClick="@{() -> presenter.onSaveClick(task)}" />
      </LinearLayout>
  </layout>
  ```
 * Listeners are represented by lambda expressions that are allowed only as root elements of your expressions. When a callback is used in an expression, Data Binding automatically creates the necessary listener and registers for the event. When the view fires the event, Data Binding evaluates the given expression. As in regular binding expressions, you still get the null and thread safety of Data Binding while these listener expressions are being evaluated.
 * 对于你的表达式，只有作为一个根元素，监听者才允许用lambda表达式表示。

* Note that in the example above, we haven't defined the view parameter that is passed into onClick(android.view.View). Listener bindings provide two choices for listener parameters: you can either ignore all parameters to the method or name all of them. If you prefer to name the parameters, you can use them in your expression. For example, the expression above could be written as:
* 注意在上面的例子中，我们还没有定义传递到onClick（View view）中的视图参数。监听者绑定队友监听参入提供两种选择：你可以忽略所有方法的参数或者他们的名字。如果你更倾向于命名参数，你可以在你的表达式中使用他们。例如，以上的表达式可以被写作：
```
  android:onClick="@{(view) -> presenter.onSaveClick(task)}"

```
* Or if you wanted to use the parameter in the expression, it could work as follows:
* 或者如果你想在你表达式中去使用参数，它可以作为如下方式运行：
```
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```
```
  android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```
* You can use a lambda expression with more than one parameter:
* 超过一个参数时你可以使用一个lambad表达式

``` 
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```
```
  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```
* If the event you are listening to returns a value whose type is not void, your expressions must return the same type of value as well. For example, if you want to listen for the long click event, your expression should return boolean.
* 如果你监听的事件返回了一个类型不是void 的值，你的表达式必须返回一样类型的值。例如，如果你想去监听长点击事件，你的表达式应该返回布尔类型。
```
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
```
```
 android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
 ``` 
 
 * If the expression cannot be evaluated due to null objects, Data Binding returns the default Java value for that type. For example, null for reference types, 0 for int, false for boolean, etc.
 * 如果表达式因为空对象不能被评估，数不绑定返回一个默认类型的Java值。例如，Null对应引用类型，0对应整型，false对应布尔类型。

* If you need to use an expression with a predicate (e.g. ternary), you can use void as a symbol.
* 如果你需要在表达式中使用断言，你可以使用void作为一个标志。
```
  android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

# Avoid Complex Listeners 避开复合监听

* Listener expressions are very powerful and can make your code very easy to read. On the other hand, listeners containing complex expressions make your layouts hard to read and unmaintainable. These expressions should be as simple as passing available data from your UI to your callback method. You should implement any business logic inside the callback method that you invoked from the listener expression.

* 监听表达式是非常强大的，它可以提高你的代码的可读性。另一方面，监听者包含的复合表达式增加你的布局的阅读难度和不可维护。对于你的回调方法的表达式应该尽可能简单就像通过你的UI的传递可用的数据给你的回调方法一样。你应该在从监听表达式调用的回调方法中事件所有的业务逻辑。

* Some specialized click event handlers exist and they need an attribute other than android: onClick to avoid a conflict. The following attributes have been created to avoid such conflicts:
* 存在一些专门的单击事件处理程序需要比android: onClick多一个其他的属性以避免冲突。以下的属性被创建去避免冲突：
```

Class	         Listener Setter	                                 Attribute
SearchView	setOnSearchClickListener(View.OnClickListener)	android:onSearchClick
ZoomControls	setOnZoomInClickListener(View.OnClickListener)	android:onZoomIn
ZoomControls	setOnZoomOutClickListener(View.OnClickListener)	android:onZoomOut
```
# Layout Details 布局详情

### Imports 

* Zero or more import elements may be used inside the data element. These allow easy reference to classes inside your layout file, just like in Java.

* 零个或者更多被导入的元素可以在数据元素里面使用。在你的布局文件里可以就像在Java里 面一样很容易的引用类。

```
<data>
    <import type="android.view.View"/>
</data>
```
* Now, View may be used within your binding expression:

* 现在，可以在你的绑定表达式里面使用View
```
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```
* When there are class name conflicts, one of the classes may be renamed to an "alias:"

* 当类名冲突时，一个类可以使用别名。
```
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

* Now, Vista may be used to reference the com.example.real.estate.View and View may be used to reference android.view.View within the layout file. Imported types may be used as type references in variables and expressions:
* 现在，在布局文件理Vista可以被使用去引用 com.example.real.estate.View并且View也可以被使用去引用com.example.real.estate.View。在变凉和表达式中导入类型可以被作为类型引用去使用。
```
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User&gt;"/>
</data>
```
* Note: Android Studio does not yet handle imports so the autocomplete for imported variables may not work in your IDE. Your application will still compile fine and you can work around the IDE issue by using fully qualified names in your variable definitions.
* 注意：Android Studio至今不能处理导入，所以在你的IDE中自动导入变量不能工作。您的应用程序仍然可以很好地编译，你可以通过在变量定义使用完全合格的名称解决此问题的IDE。

```
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
 ```
 * Imported types may also be used when referencing static fields and methods in expressions:

* 当用引用静态字段和方法时，也可以在表达式中使用导入的类型。
```
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
 ```
 * Just as in Java, java.lang.* is imported automatically.

* 就像在Java中一样，java.lang.*是自动导入的
# Variables
* Any number of variable elements may be used inside the data element. Each variable element describes a property that may be set on the layout to be used in binding expressions within the layout file.
* 任何变量元素的成员都可以在数据元素里面使用。每一个变量元素描述一个属性，该属性可以在布局文件的布局里通过使用绑定表达式设置。
```
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```
* The variable types are inspected at compile time, so if a variable implements Observable or is an observable collection, that should be reflected in the type. If the variable is a base class or interface that does not implement the Observable* interface, the variables will not be observed!
* 变量类型在编译时检查，所以如果一个变量实现了Observable或者是一个obsevable集合，在类型中该变量应该被反射。如果该变凉是一个基本类型或者没有实现Observable及其子类接口的接口，该变量就不能被观察！

* When there are different layout files for various configurations (e.g. landscape or portrait), the variables will be combined. There must not be conflicting variable definitions between these layout files.
* 当在不同的布局文件中且各个布局文件的配置也不同时，该变凉将被整合。在那些布局文件之间变量的定义绝对不能有冲突。

* The generated binding class will have a setter and getter for each of the described variables. The variables will take the default Java values until the setter is called — null for reference types, 0 for int, false for boolean, etc.
* 生成的绑定类对于每一个描述的变量将会生成一个setter和getter构造器。在setter方法被调用之前该变量将获得一个默认的Java值－引用是null，整型是0，布尔类型是false

* A special variable named context is generated for use in binding expressions as needed. The value for context is the Context from the root View's getContext(). The context variable will be overridden by an explicit variable declaration with that name.
* 为了使用绑定表达式中，生成一个特殊的被成为上下文的变量是必须的。该上下文的值就是来自于根视图中通过getContext（）方法获得的上下文。该上下文变量将被一个使用名字显示声明的变量覆盖。

# Custom Binding Class Names 自定义绑定类名字
* By default, a Binding class is generated based on the name of the layout file, starting it with upper-case, removing underscores ( _ ) and capitalizing the following letter and then suffixing "Binding". This class will be placed in a databinding package under the module package. For example, the layout file contact_item.xml will generate ContactItemBinding. If the module package is com.example.my.app, then it will be placed in com.example.my.app.databinding.
* 默认的，一个绑定类是基于布局文件的名字进行生成，使用大写开始，移除下划线（_）并且后面跟着的字大写然后添加Binding后缀。该类奖杯放置在模块包下的databinding包中。例如，布局文件contact_item.xml将生成ContactItemBinding.如果模块包是com.example.myapp，那么该类将被放置在com.example.myapp.databinding包中。
* Binding classes may be renamed or placed in different packages by adjusting the class attribute of the data element. For example:

* 绑定类的名字可以通过调整数据元素中类的属性可以重命名。例如：
```
<data class="ContactItem">
    ...
</data>
```
* This generates the binding class as ContactItem in the databinding package in the module package. If the class should be generated in a different package within the module package, it may be prefixed with ".":
* 生成的ContactItem绑定类再模块包下的databinding包中，如果希望该类在模块包下不同的包中，可以使用.前缀。
```
<data class=".ContactItem">
    ...
</data>
```
* In this case, ContactItem is generated in the module package directly. Any package may be used if the full package is provided:
* 在这种情况下，ContactItem直接生成在模块包下，如果提供完整的包，任何包都可以使用它。

```
<data class="com.example.ContactItem">
    ...
</data>

```
# Includes
* Variables may be passed into an included layout's binding from the containing layout by using the application namespace and the variable name in an attribute:
* 来自于包含布局的变量可以在一个属性中通过使用应用程序命名空间和和变量名传入一个被included进来的绑定布局中
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```
* Here, there must be a user variable in both the name.xml and contact.xml layout files.

* 这里的name.xml和contact.xml布局文件中都必须有一个user变量
* Data binding does not support include as a direct child of a merge element. For example, the following layout is not supported:

* 数据绑定不支持在作为一个merge元素的直接子元素进行include操作。
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```
# Expression Language 表达式语言
### Common Features 常用特性
* The expression language looks a lot like a Java expression. These are the same:
* 表达式语言看上去很像java表达式。下面是一些相同的部分

* Mathematical + - / * %    数学运算
* String concatenation +    字符串连接
* Logical && ||				 逻辑运算符
* Binary & | ^					位运算符
* Unary + - ! ~					一元运算符
* Shift >> >>> <<				位移运算
* Comparison == > < >= <=		比较运算
* instanceof						判断实例类型
* Grouping ()						分组
* Literals - character, String, numeric, null
* Cast
* Method calls
* Field access
* Array access []
* Ternary operator ?:
### Examples: 例子

``` 
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
# Missing Operations 没有的操作
* A few operations are missing from the expression syntax that you can use in Java.
* 一些可以在Java中使用的操作在表达式语法中是没有的

* this
* super
* new
* Explicit generic invocation
# Null Coalescing Operator Null 的聚合操作
The null coalescing operator (??) chooses the left operand if it is not null or the right if it is null.
null聚合操作(??)如果不为null选择左边的操作数，如果为null，选择右边的操作数

``` 
android:text="@{user.displayName ?? user.lastName}"
```
* This is functionally equivalent to:
* 以下的功能是等价的
```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```
#Property Reference 属性引用
* The first was already discussed in the Writing your first data binding expressions above: short form JavaBean references. When an expression references a property on a class, it uses the same format for fields, getters, and ObservableFields.
* 在上面有关JavaBean的使用部分中已经讨论了如何写你的第一个数据绑定表达式。当表达式引用了一个类的属性时，他要使用一样的fields，getters，ObservableFields

```
android:text="@{user.lastName}"
```
#Avoiding NullPointerException 避免空指针异常

* Generated data binding code automatically checks for nulls and avoid null pointer exceptions. For example, in the expression @{user.name}, if user is null, user.name will be assigned its default value (null). If you were referencing user.age, where age is an int, then it would default to 0.

* 生成的数据绑定代码会自动检查空引用和空指针异常。例如，在表达式```@{user.name}```,如果user是null，user.name将被配置一个默认的null值。如果你引用user.age,age是一个int类型字段，那么他将默认是0.

#Collections 集合

* Common collections: arrays, lists, sparse lists, and maps, may be accessed using the [] operator for convenience.
```
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String&gt;"/>
    <variable name="sparse" type="SparseArray&lt;String&gt;"/>
    <variable name="map" type="Map&lt;String, String&gt;"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```
#String Literals
* When using single quotes around the attribute value, it is easy to use double quotes in the expression:
```
android:text='@{map["firstName"]}'
```
* It is also possible to use double quotes to surround the attribute value. When doing so, String literals should either use the ' or back quote (`).
```
android:text="@{map[`firstName`}"
android:text="@{map['firstName']}"
```
#Resources

* It is possible to access resources as part of expressions using the normal syntax:
* 使用正常的表达式语法可以获得资源
```
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```
* Format strings and plurals may be evaluated by providing parameters:
* 格式化字符串和复数可以通过提供的参数进行评估：
```
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```
* When a plural takes multiple parameters, all parameters should be passed:
* 当一个复数采用多个参数是，所有的参数都应该被传入。
```

  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, 
orangeCount)}"
```
* Some resources require explicit type evaluation.
* 一些资源要求明确评估类型
* 

```
Type	           Normal Reference	   Expression Reference
String[]	        @array     	         @stringArray
int[]	            @array	             @intArray
TypedArray          @array           	 @typedArray
Animator	        @animator	         @animator
StateListAnimator   @animator	         @stateListAnimator
color int	        @color	             @color
ColorStateList	    @color	             @colorStateList
```
# Data Objects

* Any plain old Java object (POJO) may be used for data binding, but modifying a POJO will not cause the UI to update. The real power of data binding can be used by giving your data objects the ability to notify when data changes. There are three different data change notification mechanisms, Observable objects, observable fields, and observable collections.
* 数据绑定可以使用所有的Java对象（POJO），但是修改一个POJO不会引起一个UI的更新。通过给予你的数据对象一个在数据发生变化时可以通知的能力来使用数据绑定真正强大的能力。这是三个不同的数据改变通知机制，Observable对象，observable字段，observable集合
* When one of these observable data object is bound to the UI and a property of the data object changes, the UI will be updated automatically.

* 当被绑定到UI中的有一个是observable数据对象时，如果数据对象的属性值改变了，UI也会自动更新。

# Observable Objects

* A class implementing the Observable interface will allow the binding to attach a single listener to a bound object to listen for changes of all properties on that object.
* 一个实现了Observable接口的类将允许给绑定对象绑定关联一个监听器去监听该对象中所有属性的变化情况。
* The Observable interface has a mechanism to add and remove listeners, but notifying is up to the developer. To make development easier, a base class, BaseObservable, was created to implement the listener registration mechanism. The data class implementer is still responsible for notifying when the properties change. This is done by assigning a Bindable annotation to the getter and notifying in the setter.
* Observable接口有一个添加和移除监听器的机制，而已通知是在不断的发展改进的。创建一个基本的BaseObservable类区实现监听器注册机制对于开发来说是很容易的。通过分配一个Bindable注解给数据类的getter和setter，当属性改变时，数据类的视线仍然能响应通知。

```
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```
* The Bindable annotation generates an entry in the BR class file during compilation. The BR class file will be generated in the module package. If the base class for data classes cannot be changed, the Observable interface may be implemented using the convenient PropertyChangeRegistry to store and notify listeners efficiently.

* 在编译的时候，Bindable注解在BR类文件中生成一个条目。在模块包下生成一个BR文件。如果数据类是一个不能改变的类，Observable接口会通过使用适当的PropertyChangeRegister去有效的存储和通知监听者来实现。

#ObservableFields
* A little work is involved in creating Observable classes, so developers who want to save time or have few properties may use ObservableField and its siblings ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, and ObservableParcelable. ObservableFields are self-contained observable objects that have a single field. The primitive versions avoid boxing and unboxing during access operations. To use, create a public final field in the data class:
* 在创建Observable类时还是要做一些工作的，所以一些想去解约事件或者希望有一些可使用的属性的开发着可以使用ObservableField和与之类似的ObservableBoolean，ObservableByte，ObservableChar，ObservableDouble，ObservableLong，ObservableFloat，ObservableDouble，ObservableParcelable。ObservableFields 是一个有单一字段的自包含的observable对象。最初的版本是避免在操作时做装箱和拆箱操作。在数据类中以public final 字段形式创建并使用。



















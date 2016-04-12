dagger2 官方地址：http://google.github.io/dagger/

## Home

Dagger 是为 Java 和 Android 设计的静态编译期注入框架。是为了替换由 Square 公司开发的早期版本 dagger1， 目前由谷歌维护。

Dagger 旨在解决之前基于反射的解决方案所带来的开发和性能问题。更多细节可查看由+Gregory Kick 推出的视频 [DAGGER 2 - A New Type of dependency injection](https://www.youtube.com/watch?v=oK_XtfXPkqw)。

## 用户手册

一个项目中最重要的类有这些， 比如 BarcodeDecoder， KoopaPhysicsEngine， AudioStreamer 等等。 这些类也许会需求这些依赖， 如一个 BarcodeCameraFinder， DefaultPhysicsEngine 和 HttpStreamer。 
相比而言， 这些类就显得占了位置却不起什么作用， 如 BarcodeDecoderFactory， CameraServiceLoader， MutableContextWrapper 等等。 这些类就如同拙劣的粘着剂将重要的类联系在一起。
Dagger就是为了解决依赖注入模式中， 编写这些没有实际作用的工厂类模板文件的负担。 它使你把精力投入到更有意义的编码工作中， 例如声明依赖， 提供依赖， 然后走起你的 app ~
基于标准的 javax。inject (JSR 330)注解库， 每个类将会很容易测试。 你不再需要仅仅为了替换一个业务 Service ，而写一堆没用的模板文件。 

### Dagger2 有何不同
Dagger 构造你的应用程序的类对象并自动注入它们的依赖。 它使用 javax。inject。Inject 下的注解， 来标记哪些构造器或者全局变量需要 Dagger 注意。

使用 @Inject 来标记一个构造器， 表示 Dagger 指定此构造器来构造相应类的实例。 当某处需要这个类的实例， Dagger 将自动调用此构造器构造实例， 并注入到对应位置。 如果构造器有参数要求， Dagger 也会查找需要的参数实例注入进去。 

```
// 热虹吸抽水器
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) { // 需要 Heater 实例，Dagger 自动查找并注入
    this.heater = heater;
  }

  ...
}
```
使用 @Inject 直接标记一个全局变量， 在下面的示例中， Dagger 自动查找 Heater 和 Pump 的实例并注入

```
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  。。。
}
```

如果某个类只有全局变量被标记了 @Inject， 但构造器没有被标记， 那么 Dagger 会注入这些全局变量， 但不再构造该类的实例(?)。 使用 @Inject 来标记一个无参构造器来表明 Dagger 也会构造它的实例。 
Dagger 也支持方法注入， 但是往往偏爱构造器或全局变量的注入。
没有 @Inject 注解的类， 将不会被 Dagger 创建。

### Satisfying Dependencies 满足依赖
默认的， 如之前所述， Dagger 通过构造相应的类型来满足每个标注的依赖。 比如当你需要一个 CoffeeMaker 实例， Dagger 会调用 new CoffeeMaker() 并注入好它的 @Inject 标记的全局变量来获取一个 CoffeeMaker 实例。
但是 @Inject 并非万能:
- 接口无法构造实例
- 第三方类库无法被注解
- 配置相关的对象必须已经初始化好(Configurable objects must be configured!)

在以下这些场景中， 使用 @Inject 是多余且别扭的， 应该使用 @Provides 来标注一个方法来满足对应的依赖， 这个方法的返回值标注了它提供的类型， 也就是它可以响应哪一种类型的依赖请求。 
如下面这个案例， 当请求一个 Heater 对象时， provideHeater() 方法会被调用

```
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```

@Provides 注解的方法也可以有它的依赖请求: 

```
@Provides static Pump providePump(Thermosiphon pump) {
  return pump;
}
```

所有 @Provides 注解的方法必须属于一个 module， 也就是一个被 @Module 标记的类:

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }

  @Provides static Pump providePump(Thermosiphon pump) {
    return pump;
  }
}
```
按照惯例， @Provides 注解的方法都应该以 "provide" 为前缀， @Module 注解的类都应该以 "module" 为后缀。

### 构建图容器

这些拥有 @Inject 和 @Provides 注解的类会组成一个对象图， 通过他们的依赖连接起来。 调用代码就像 main 方法， 通过定义好的一些根节点， 访问这个对象图。 在 Dagger2 中， 这些根节点由一个拥有一些没有参数且返回需要的类型的方法构成的接口所定义。 

通过 @Component 标注这样的接口， 并将一些 module 类赋值给 @Component 注解的 modules 参数， Dagger2 会依据这些接口来生成它们的实现类。

```
@Component(modules = DripCoffeeModule。class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

实现类的类名是接口名称加上 "Dagger" 的后缀， 调用它的 builder() 静态方法， 使用其返回的建造者实例， 设置依赖， 再调用 build() 来构建一个新的实例。 

```
CoffeeShop coffeeShop = DaggerCoffeeShop。builder()
    。dripCoffeeModule(new DripCoffeeModule())
    。build();
```

提示: 如果 @Component 标记的类不是顶层类(是一个内部类)， 那么生成的实现类名将带上它的外部类名， 使用下划线分隔开。 例如:

```
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```

生成的类名是 DaggerFoo_Bar_BazComponent
任何一个自带可访问默认构造器的 module 类， 会被禁用， 由其建造方法取代:

```
CoffeeShop coffeeShop = DaggerCoffeeShop。create();
```

对于一个所有的 @Provides 的方法都是静态的 module 类， 就无需构造它的实例; 如果所有的依赖都无需由用户创建， 那么实现类依然会提供 create() 方法， 可用于获取实例

然后就可以很方便的使用 Dagger 生成的实现来获取一个完全配置并可用的实例:

```
public class CoffeeApp {
  public static void main(String[] args) {
    CoffeeShop coffeeShop = DaggerCoffeeShop。create();
    coffeeShop。maker()。brew();
  }
}
```

### 在对象图中绑定
上面的例子演示了如何使用一些典型的绑定构造一个 component， 但还有多种机制来形成向对象图的绑定; 下面的内容可以作为依赖和生成完整格式的 component:

- 通过 @Component。modules 或 @Module。includes 直接引用的 Module 类声明的 @Provides 方法
- 拥有一个 @Inject 注解的构造器的类， 没有限制作用域， 或者有一个 @Scope 注解且对应着 component
- component 依赖的组件提供方法
- component 本身
- 所有包含的 subcomponent 的不符合的建造器
- 以上绑定的 Provider 或懒加载包装器
- 任何类型的 MembersInjector

### 单例和作用域绑定
使用 @Singleton 注解一个 @Provides 注解的方法或可以注入的类， 对象图会对其客户端使用单例模式:

```
@Provides @Singleton static Heater provideHeater() {
  return new ElectricHeater();
}
```

标记在可注入的类上的 @Singleton 注解也可以作为文档注解。 提醒了维护者这个类可能共享在多个线程。

```
@Singleton
class CoffeeMaker {
  。。。
}
```

由于 Dagger2 关联了对象图定义了作用域的实例与 component 的实现实例， components 需要声明它们所代表的作用域。 例如在一个 component 中使用了一个单例模式的绑定， 又使用了另外某个作用域绑定， 这就没有意义了。 因为这些作用域拥有不同的生命周期并因此处在不同的拥有不同生命周期的 component。 声明一个关联了指定的作用域的 component， 只要添加一个 @Scope 注解。 

```
@Component(modules = DripCoffeeModule。class)
@Singleton
interface CoffeeShop {
  CoffeeMaker maker();
}
```

### 懒注入
有时候你需要一个类延迟实例化， 比如一个 T 类型的绑定， 你可以使用 Lazy<T> 类型， 推迟实例化直到 Lazy<T> 的 get 方法被调用。 如果 T 是单例， 那么在对象图中， 所有的注入的地方获取的都是同一个对象， 否则每个注入都会获取自己的实例。 之后对于每个指定 Lazy<T> 调用 get 方法都会返回同样的 T 实例:

```
class GridingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to 。get() and cached。
      lazyGrinder。get()。grind();
    }
  }
}
```

### 提供器注入

有时候你需要多个实例而不是仅仅一个值， 这时候你也许有多种选择， 例如工厂， 建造者等等。 一个选择就是注入 Provider<T> 类型而不是 T， 每当 get 方法被调用， 它都会会调用绑定逻辑。 如果绑定逻辑是使用 @Inject 注解构造器， 新的实例会被构造， 但是一个 @Provides 注解的方法并不能确保是这样。 

```
class BigCoffeeMaker {
  @Inject Provider<Filter> filterProvider;

  public void brew(int numberOfPots) {
  。。。
    for (int p = 0; p < numberOfPots; p++) {
      maker。addFilter(filterProvider。get()); //new filter every time。
      maker。addCoffee(。。。);
      maker。percolate();
      。。。
    }
  }
}
```

提示: 注入 Provider<T> 类型会导致一些令人迷惑的代码， 并在你的对象图中有些混淆作用域和架构的意思。 经常的你会希望使用工厂， Lazy<T> 或重新组织了你的代码的生命期和结构， 以使得可以注入一个类型 T。 但有些场景下 Provider<T> 也会成为你的救命稻草。 一个通常的用法就是当你必须要用一个陈旧的架构， 并没有按你的对象的本来的生命周期规整。 比如 servlet 设计就是单例， 但仅在请求域中是有效的。 

### 标识器 Qualifiers
有时一个单独的类不足以识别一个依赖。 比如一个成熟的咖啡机 app 也许为了加热水和加热盘子， 会需要不同的 Heater 对象。 
在这样的场景下， 我们添加了标识器(qualifier)注解。 任何注解都可以拥有一个 @Qualifier 注解， 下面是 @Named 注解的声明， 一个包含在 javax。inject 中的标识器注解:

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {
  String value() default "";
}
```

你可以创建自己的标识器注解， 或者也可以用 @Named。 可以注解成员变量或参数。 类型和标识器注解共同标记一个依赖:

```
class ExpensiveCoffeeMaker {
  @Inject @Named("water") Heater waterHeater;
  @Inject @Named("hot plate") Heater hotPlateHeater;
  。。。
}
```
设置标识器的值并设置在 @Provides 注解的方法上:

```
@Provides @Named("hot plate") static Heater provideHotPlateHeater() {
  return new ElectricHeater(70);
}

@Provides @Named("water") static Heater provideWaterHeater() {
  return new ElectricHeater(93);
}
```

依赖可能不能拥有多个标识器注解。 

### 编译器校验
Dagger 注解处理机制是严格的， 并且当出现无效或不完整的绑定会报编译错误。 例如这个 module 装载在一个缺少 Executor 的绑定的 component :

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater(Executor executor) {
    return new CpuHeater(executor);
  }
}
```
编译期会报异常:

```
[ERROR] COMPILATION ERROR :
[ERROR] error: java。util。concurrent。Executor cannot be provided without an @Provides-annotated method。
```

解决这个错误只要在 component 下添加一个 @Provides 注解的返回类型是 Executor 的方法。 @Inject， @Module 和 @Provides 这些注解都会独立验证， 所有绑定关系的验证都发生在 component 层级上。 Dagger 1 是依靠在 Module 层级上的严格验证， 但是 Dagger 2 为了更完整的对象图层面上的验证， 省略了这些过程。

### 编译器代码生成
Dagger 的注解处理机会自动生成一些命名诸如 CoffeeMaker\_Factory。java 或者 CoffeeMaker\_MembersInjector。java 的源代码文件， 这些文件就是 Dagger 实现的细节。 你不需要直接使用它， 尽管你很容易在注入的时候在其内部单步调试。 这些生成的类中， 唯一你要关心的是哪些组件名加上 "Dagger" 前缀的哪些类。

### 在构建中使用 Dagger2

你需要在你的项目环境中引入 dagger-2。0。jar。 为了能够实现生成代码的功能， 还需要在编译环境下引入 dagger-compiler-2。0。jar。

在一个 Maven 项目中:

```
<dependencies>
  <dependency>
    <groupId>com。google。dagger</groupId>
    <artifactId>dagger</artifactId>
    <version>2。0</version>
  </dependency>
  <dependency>
    <groupId>com。google。dagger</groupId>
    <artifactId>dagger-compiler</artifactId>
    <version>2。0</version>
    <optional>true</optional>
  </dependency>
</dependencies>
```

## Android

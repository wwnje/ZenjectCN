# 绑定（Binding）
每个依赖注入框架最终只是一个将类型绑定到实例的框架。

在Zenject中，依赖关系映射(dependency mapping )是通过向称为容器(container)的东西添加绑定来完成的。然后，容器应该通过递归地解析给定对象的所有依赖项来“知道”如何在应用程序中创建所有对象实例。

当容器(container)被要求构造给定类型的实例时，它使用C＃反射(reflection)来查找构造函数参数列表以及使用[Inject]属性标记的所有字段/ 属性。然后，它尝试解析每个必需的依赖项，用于调用构造函数并创建新实例。

因此，每个Zenject应用程序必须告诉容器(container)如何解决这些依赖关系，这是通过Bind命令完成的。例如，给定以下类：
```
public class Foo
{
    IBar _bar;

    public Foo(IBar bar)
    {
        _bar = bar;
    }
}
```
您可以使用以下内容连接此类的依赖项：
```
Container.Bind<Foo>().AsSingle();
Container.Bind<IBar>().To<Bar>().AsSingle();
```
这告诉Zenject每个需要类型为Foo的依赖项的类应该使用相同的实例，它将在需要时自动创建。同样地，任何需要IBar接口的类（如Foo）都将被赋予Bar类型的相同实例。

bind命令的完整格式如下。请注意，在大多数情况下，您不会使用所有这些方法，并且在未指定时它们都具有逻辑默认值
```
Container.Bind<ContractType>()
    .WithId(Identifier)
    .To<ResultType>()
    .FromConstructionMethod()
    .AsScope()
    .WithArguments(Arguments)
    .OnInstantiated(InstantiatedCallback)
    .When(Condition)
    .(Copy|Move)Into(All|Direct)SubContainers()
    .NonLazy()
    .IfNotBound();
```

解析:(Where)
* **ContractType** = 您要为其创建绑定的类型
    * 该值将对应于正在注入的字段/参数的类型

* **ResultType** = 要绑定的类型
    * 默认值：**ContractType**
    * 此类型必须等于**ContractType**或从**ContractType**派生。如果未指定，则假定为ToSelf()，这意味着**ResultType**将与**ContractType**相同。此值将由作为**ConstructionMethod**提供的任何内容使用，以检索此类型的实例

* **Identifier** = 用于唯一标识绑定的值。在大多数情况下，这可以忽略，但在需要区分具有相同合同类型的多个绑定的情况下，这可能非常有用。详情请见[此处](Identifiers.md)。

* **ConstructionMethod** = 创建/检索ResultType实例的方法。有关各种构造方法的更多详细信息，请参阅[此处](construction_methods.md)。
    * 默认值: FromNew()
    * 例子: eg. FromGetter, FromMethod, FromResolve, FromComponentInNewPrefab, FromSubContainerResolve, FromInstance等.
    
* 范围(**Scope**) = 此值确定在多次进样中(re-used across)重复使用生成的实例的频率（或根本不存在）。

    * 默认值: **AsTransient**。但请注意，并非所有绑定都具有默认值，因此如果未提供，则会抛出异常。不需要显式设置范围的绑定是与构造方法的任何绑定，而构造方法是搜索而不是从头创建新对象（例如FromMethod，FromComponentX，FromResolve等）
    * 它可以是以下之一:
        * a.**AsTransient** - 根本不会重复使用该实例。每次请求ContractType时，DiContainer将再次执行给定的构造方法
        * b.**AsCached** - 每次请求ContractType时都会重复使用ResultType的相同实例，它会在第一次使用时延迟(lazily)生
        * c.**AsSingle** - 与AsCached完全相同，只是如果已存在ResultType绑定，它有时会抛出异常。它只是一种确保给定ResultType在容器中唯一的方法。但请注意，它只能保证给定容器中只有一个实例，这意味着在子容器(sub-container)中使用具有相同绑定的AsSingle可以生成第二个实例。
    * 在大多数情况下，您可能只想使用**AsSingle**，但是AsTransient和AsCached也有它们的用途。
* **Arguments** = 构造ResultType类型的新实例时要使用的对象列表。这可以用作为表单中的参数添加其他绑定的替代方法
`Container.BindInstance(arg).WhenInjectedInto<ResultType>()`
* **InstantiatedCallback** = 在某些情况下，能够在实例化后自定义对象很有用(In some cases it is useful to be able customize an object after it is instantiated.)。特别是，如果使用第三方库，可能需要更改其中一种类型的几个字段。对于这些情况，您可以将方法传递给OnInstantiated，以便自定义新创建的实例。例如：
```
Container.Bind<Foo>().AsSingle().OnInstantiated<Foo>(OnFooInstantiated);
void OnFooInstantiated(InjectContext context, Foo foo)
{
    foo.Qux = "asdf";
}
```
或者，等效地：
```
Container.Bind<Foo>().AsSingle().OnInstantiated<Foo>((ctx, foo) => foo.Bar = "qux");
```
请注意，您也可以使用FromFactory绑定自定义工厂，直接调用Container.InstantiateX，然后再调整它以获得相同的效果，但OnInstantiated在某些情况下可能更容易

* **Condition** = 要选择此绑定必须为true的条件。有关详细信息，请参见[此处](conditional_bindings.md)

* (**Copy|Move**)Into(**All|Direct**)SubContainers = 99％的用户可以忽略此值。它可用于自动拥有子容器(subcontainers)继承的绑定。例如，如果您有一个类Foo并且您希望将一个唯一的Foo实例自动放置在容器和每个子容器(subcontainer)中，那么您可以添加以下绑定：
`Container.Bind<Foo>().AsSingle().CopyIntoAllSubContainers()`
换句话说，结果将等同于将Container.Bind<Foo>().AsSingle()语句复制并粘贴到每个子容器的安装程序中。

或者，如果您只想在子容器中使用Foo而不是当前容器：
`Container.Bind<Foo>().AsSingle().MoveIntoAllSubContainers()`

或者，如果您只希望Foo成为直接子子容器(child subcontainer)，而不是这些子容器的子容器(subcontainers of these subcontainers)：
`Container.Bind<Foo>().AsSingle().MoveIntoDirectSubContainers()`

* **NonLazy** = 通常，ResultType仅在首次使用绑定时进行实例化（也称为“懒惰”("lazily")）。但是，使用NonLazy时，将立即在启动时创建ResultType。

* **IfNotBound** = 当这个被添加到绑定(binding)并且已经与给定的contract type + identifier绑定时，将跳过此绑定(binding)。
(When this is added to a binding and there is already a binding with the given contract type + identifier, then this binding will be skipped.)








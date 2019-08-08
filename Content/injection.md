# Injection

有许多不同的方法来声明容器的依赖关系，这些方法在[下一节中](binding.md)有介绍。 还有几种方法可以将这些依赖项注入到类中。 这些是：

1  -  **构造函数注入**(Constructor Injection)

```csharp
public class Foo
{
    IBar _bar;

    public Foo(IBar bar)
    {
        _bar = bar;
    }
}
```

2  -  **字段注入**(Field Injection)

```csharp
public class Foo
{
    [Inject]
    IBar _bar;
}
```

在调用构造函数后立即进行字段注入。 标记为`[Inject]`属性的所有字段都在容器中查找并给出一个值。 请注意，这些字段可以是私有或公共字段，但仍会发生注入。

3  -  **属性注入**(Property Injection)

```csharp
public class Foo
{
    [Inject]
    public IBar Bar
    {
        get;
        private set;
    }
}
```

除了应用于C＃属性之外，属性注入与字段注入相同。 就像字段一样，在这种情况下，setter可以是私有的或公共的。

4  -  **方法注入**(Method Injection)

```csharp
public class Foo
{
    IBar _bar;
    Qux _qux;

    [Inject]
    public Init(IBar bar, Qux qux)
    {
        _bar = bar;
        _qux = qux;
    }
}
```

方法注入与构造函数注入非常相似。

请注意以下事项：

 - 注入方法是MonoBehaviours的推荐方法，因为MonoBehaviours不能有构造函数。

 - 可以有任意数量的注入方法。在这种情况下，它们按基础Base类到派生Derived类的顺序调用。这有助于避免需要通过构造函数参数将派生类中的许多依赖项转发到基类，同时还保证基类注入方法首先完成，就像构造函数的工作原理一样。

 - 在所有其他注入类型之后调用注入方法。它以这种方式设计，以便这些方法可用于执行初始化逻辑，这可能会使用注入的字段或属性。另请注意，如果您只想执行某些初始化逻辑，则可以将参数列表保留为空。

 - 您可以放心地假设您通过注入方法接收的依赖项本身已经被注入（唯一的例外是在您具有循环依赖项的情况下）。如果使用注入方法执行一些基本初始化，这可能很重要，因为在这种情况下，您可能还需要初始化给定的依赖项

 - 但请注意，对初始化逻辑使用注入方法通常不是一个好主意。通常最好使用`IInitializable.Initialize`或`Start()`方法，因为这将允许首先创建整个初始对象图(object graph)

**建议**

与constructor/method)注入相比，最佳实践是更推荐field/property注入。
* 构造函数注入强制依赖只能在类创建时解析一次，这通常是您想要的。在大多数情况下，您不希望为初始依赖项公开公共属性，因为这表明它可以更改。

* 构造函数注入保证类之间没有循环依赖，这通常是一件坏事。当使用其他注入类型时，Zenject确实允许循环依赖，例如方法/字段/属性注入(method/field/property injection)

* 如果您决定在没有DI框架（如Zenject）的情况下重新使用代码，则构造函数/方法注入更具可移植性。您可以对公共属性执行相同操作，但它更容易出错（忘记初始化一个字段并使对象处于无效状态更容易）

* 最后，构造函数/方法注入清楚地表明当另一个程序员正在读取代码时，类的所有依赖关系。他们可以简单地查看方法的参数列表。这也很好，因为当一个类有太多依赖关系时会更明显，因此应该拆分（因为它的构造函数参数列表看起来很长）
# 理论

以下是从我的角度对依赖注入(Dependency Injection)的一般概述。虽然很简单基础，所以我强烈建议你寻求其他资源以获得有关该主题的更多信息，因为还有许多其他人（通常具有更好的写作能力）已经写了关于它背后的理论。

在编写单个类以实现某些功能时，可能需要与系统中的其他类进行交互以实现其目标。 一种方法是通过调用具体的构造函数让类本身创建其依赖项：

```
public class Foo
{
    ISomeService _service;

    public Foo()
    {
        _service = new SomeService();
    }

    public void DoSomething()
    {
        _service.PerformTask();
        ...
    }
}
```

这适用于小型项目，但随着项目的增长，它开始变得笨拙。 类Foo与类'SomeService'紧密耦合。 如果我们稍后决定使用不同的具体实现，那么我们必须回到Foo类来改变它。

在考虑了这一点之后，通常你会意识到最终，Foo不应该为选择服务的具体实现细节而烦恼。 所有Foo应该关心的是履行自己的具体职责。 只要服务满足Foo要求的抽象接口，Foo就很高兴。 我们的类成为：

```
public class Foo
{
    ISomeService _service;

    public Foo(ISomeService service)
    {
        _service = service;
    }

    public void DoSomething()
    {
        _service.PerformTask();
        ...
    }
}
```
这是更好的，但现在无论什么类创建Foo（让我们称之为Bar）都有填充Foo的额外依赖项的问题：

```
public class Bar
{
    public void DoSomething()
    {
        var foo = new Foo(new SomeService());
        foo.DoSomething();
        ...
    }
}
```

而Bar类可能也并不真正关心SomeService Foo使用的具体实现。 因此，我们再次更新依赖：

```
public class Bar
{
    ISomeService _service;

    public Bar(ISomeService service)
    {
        _service = service;
    }

    public void DoSomething()
    {
        var foo = new Foo(_service);
        foo.DoSomething();
        ...
    }
}
```

因此，我们发现，在应用程序的“对象图”(object graph)中进一步推进决定使用哪些类的具体实现的责任是有用的。考虑到这一点，我们到达应用程序的入口点，此时必须满足所有依赖项才能开始。 这部分应用程序的依赖注入术语称为“组合根”(composition root)。 它通常看起来像这样：

```
var service = new SomeService();
var foo = new Foo(service);
var bar = new Bar(service);
var qux = new Qux(bar);

.. etc.
```

诸如Zenject之类的DI框架只是帮助自动化创建和分发所有这些具体依赖项的过程，因此您不需要像上面的代码那样自己明确地这样做。
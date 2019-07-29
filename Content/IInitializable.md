# IInitializable

如果您需要在给定对象上进行某些初始化(initialization)，可以在构造函数中使用初始化代码。但是，这意味着初始化逻辑将出现在正在构造的对象图的中间，因此它可能不是理想的。

更好的选择是实现`IInitializable`，然后在`Initialize()`方法中执行初始化逻辑。 然后，在安装程序(installer)中将其连接起来：

```
Container.Bind<IInitializable>().To<Foo>().AsSingle();
```

或者，如果您不想总是记住您的类实现了哪些接口，则可以使用此处描述的[快捷方式](BindInterfacesTo-and-BindInterfacesAndSelfTo.md).

`Foo.Initialize`方法将在整个对象图(object graph)构造后并调用所有构造函数之后调用。

请注意，在Unity的Awake事件期间调用初始对象图的构造函数，并且在Unity的Start事件中立即调用`IInitializable.Initialize`方法。因此，使用`IInitializable`而不是构造函数更符合Unity自己的建议，这些建议为使用Awake阶段来设置对象引用，并将Start阶段用于更复杂的初始化逻辑。

这也可能比使用构造函数或`[Inject]`方法更好，因为初始化顺序(initialization order )可以与`ITickable`类似的方式进行自定义，如[此处](update-initialization-order.md)所述。

```
public class Ship : IInitializable
{
    public void Initialize()
    {
        // Initialize your object here
    }
}
```

`IInitializable`适用于启动初始化(start-up initialization)，但是对于通过工厂动态创建的对象呢？（请参阅本节我在这里所[引用](creating-objects-dynamically.md)的内容）。 对于这些情况，您很可能希望使用`[Inject]`方法或在创建对象后调用的显式`Initialize`方法。 例如：

```
public class Foo
{
    [Inject]
    IBar _bar;

    [Inject]
    public void Initialize()
    {
        ...
        _bar.DoStuff();
        ...
    }
}
```


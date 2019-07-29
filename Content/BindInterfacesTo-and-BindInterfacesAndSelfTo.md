# BindInterfacesTo and BindInterfacesAndSelfTo

如果你最终使用如上所述的`ITickable`，`IInitializable`和`IDisposable`接口，你通常会得到这样的代码：

```
Container.Bind(typeof(Foo), typeof(IInitializable), typeof(IDisposable)).To<Foo>().AsSingle();
```

有时这可能有点冗长。 此外，它并不理想，因为如果我后来决定Foo不需要`Tick()`或`Dispose()`，那么我必须保持安装程序(installer)同步。

一个更好的想法可能是只使用这样的接口：

```
Container.Bind(new[] { typeof(Foo) }.Concat(typeof(Foo).GetInterfaces())).To<Foo>().AsSingle();
```

这种模式足够有用，Zenject包含一个自定义绑定方法。 上面的代码相当于：


```
Container.BindInterfacesAndSelfTo<Foo>().AsSingle();
```

现在，我们可以添加和删除Foo的接口，并且安装程序(installer)保持不变。

在某些情况下，您可能只想绑定接口，并将Foo与其他类隐藏起来。 在这种情况下，您将使用BindInterfacesTo方法：

```
Container.BindInterfacesTo<Foo>().AsSingle()
```

在这种情况下，将扩展到：

```
Container.Bind(typeof(IInitializable), typeof(IDisposable)).To<Foo>().AsSingle();
```
# IDisposable

如果您有应用程序关闭时要清理的外部资源，场景发生更改，或者出于上下文对象被销毁的原因，您可以将您的类声明为`IDisposable`，如下所示：

```
public class Logger : IInitializable, IDisposable
{
    FileStream _outStream;

    public void Initialize()
    {
        _outStream = File.Open("log.txt", FileMode.Open);
    }

    public void Log(string msg)
    {
        _outStream.WriteLine(msg);
    }

    public void Dispose()
    {
        _outStream.Close();
    }
}
```

然后在安装程序(installer)中，您可以包括：

```
Container.Bind(typeof(Logger), typeof(IInitializable), typeof(IDisposable)).To<Logger>().AsSingle();
```

Or you can use the [BindInterfaces shortcut](BindInterfacesTo-and-BindInterfacesAndSelfTo.md):

```
Container.BindInterfacesAndSelfTo<Logger>().AsSingle();
```

这是因为当场景改变或你的unity程序关闭时，在所有MonoBehaviours上调用unity事件`OnDestroy()`，包括SceneContext类，然后在绑定到`IDisposable`的所有对象上触发`Dispose()`。

您还可以实现类似于`ILateTickable`的`ILateDisposable`接口，因为它将在触发所有`IDisposable`对象后调用。 但是，在大多数情况下，如果顺序有问题，您最好设置一个明确的[执行顺序](update-initialization-order.md)。


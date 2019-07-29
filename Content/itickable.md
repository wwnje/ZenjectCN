# ITickable

在某些情况下，最好避免使用MonoBehaviour，而换成使用普通的C＃类。 Zenject允许您通过提供的接口来更轻松地完成此操作，这些功能之前通常需要使用MonoBehaviour。

例如，如果您有需要每帧运行的代码(例如MonoBehaviour中的Update)，那么您可以实现ITickable接口：

```csharp
public class Ship : ITickable
{
    public void Tick()
    {
        // Perform per frame tasks
    }
}
```

然后，在安装程序(installer)中将其连接起来：

```csharp
Container.Bind<ITickable>().To<Ship>().AsSingle();
```

或者，如果您不想总是记住您的类实现了哪些接口，则可以使用[此处](indInterfacesTo-and-BindInterfacesAndSelfTo.md)描述的快捷方式

请注意，所有ITickables调用Tick()的顺序也是可配置的，如[此处](update--initialization-order.md)所述。

另请注意，有接口`ILateTickable`和`IFixedTickable`，它们对应了Unity的LateUpdate和FixedUpdated方法。
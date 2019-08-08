# Using the Unity Inspector To Configure Settings

将大多数代码编写为普通C＃类而不是`MonoBehaviour`的一个含义是，您无法使用inspector在其上配置数据。 但是，您仍然可以使用以下模式在Zenject中利用此功能：

```csharp
public class Foo : ITickable
{
    readonly Settings _settings;

    public Foo(Settings settings)
    {
        _settings = settings;
    }

    public void Tick()
    {
        Debug.Log("Speed: " + _settings.Speed);
    }

    [Serializable]
    public class Settings
    {
        public float Speed;
    }
}
```

然后，在安装程序中：

```cssharp
public class TestInstaller : MonoInstaller<TestInstaller>
{
    public Foo.Settings FooSettings;

    public override void InstallBindings()
    {
        Container.BindInstance(FooSettings);
        Container.BindInterfacesTo<Foo>().AsSingle();
    }
}
```

或者，等效地：

```cssharp
public class TestInstaller : MonoInstaller<TestInstaller>
{
    public Foo.Settings FooSettings;

    public override void InstallBindings()
    {
        Container.BindInterfacesTo<Foo>().AsSingle().WithArguments(FooSettings);
    }
}
```

现在，如果我们运行场景，我们可以更改速度值以实时调整Foo类。

另一种（可以说是更好的）方法是使用`ScriptableObjectInstaller`而不是`MonoInstaller`，它具有额外的优势，你可以在运行时更改你的设置，并在播放模式停止时自动保持这些更改。 有关详细信息，请参见[此处](scriptableobject-installer.md)
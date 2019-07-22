# 安装程序 Installers

通常，每个子系统都有一些相关绑定的集合，因此将这些绑定组合成一个可重用的对象是有意义的。在Zenject中，这个可重用的对象称为“安装程序”（'installer'）。您可以按如下方式定义新安装程序：

```csharp
public class FooInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<Bar>().AsSingle();
        Container.BindInterfacesTo<Foo>().AsSingle();
        // etc...
    }
}
```

您可以通过覆盖InstallBindings方法来添加绑定，该方法由`Context`添加的安装程序调用（通常是`SceneContext`）。MonoInstaller是一个MonoBehaviour，因此您可以通过将其附加到GameObject来添加FooInstaller。由于它是GameObject，您还可以向其添加公共成员，以便从Unity检查器配置安装程序。这允许您在场景中添加引用，引用资源或简单地调整数据（有关调整数据的更多信息，请参见[此处](using-the-unity-inspector-to-configure-settings.md))。

请注意，为了触发安装程序，必须将其附加到`SceneContext`对象的Installers属性(Installers property)。安装程序按给定的顺序`SceneContext`安装（首先是脚本化对象安装程序(scriptable object installers)，然后是mono installers，然后prefab installers）但是这个顺序通常不重要（因为在安装过程中不应该实例化任何内容）。

在许多情况下，您希望安装程序派生自MonoInstaller，以便您可以进行检查器设置(inspector settings)。还有一个简单的基类`Installer`，您可以在不需要MonoBehaviour的情况下使用它。

您也可以从其他安装程序调用安装程序。例如：


```csharp
public class BarInstaller : Installer<BarInstaller>
{
    public override void InstallBindings()
    {
        ...
    }
}

public class FooInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        BarInstaller.Install(Container);
    }
}
```

请注意，在这种情况下，BarInstaller是类型`Installer<>`（注意通用参数）而不是MonoInstaller，这就是为什么我们可以简单地调用`BarInstaller.Install(Container)`并且不要求将BarInstaller添加到我们的场景中。对BarInstaller.Install的任何调用都将立即实例化BarInstaller的临时实例，然后在其上调用InstallBindings。对于此安装程序安装的所有安装程序(any installers)，这将重复此操作。还要注意，在使用`Installer<>`基类时，我们总是必须将自己作为泛型参数传递给`Installer<>`。这是必要的，以便`Installer<>`基类可以定义静态方法`BarInstaller.Install`。它也以这种方式设计，以支持运行时参数（runtime parameters）（如下所述）。

我们使用安装程序的主要原因之一就是使每个场景都能同时为所有绑定声明，这是为了使它们可以重复使用。这对于`Installer<>`类型的安装程序来说不是问题，因为你可以像上面所描述的那样为你想要使用它的每个场景调用`FooInstaller.Install`，但那么我们如何在多个场景中重用MonoInstaller呢？

有三种方法可以做到这一点.

1. **场景中的预制实例**. 将MonoInstaller附加到场景中的游戏对象后，您可以创建一个预制件。这很好，因为它允许您跨场景共享您在MonoInstaller上的检查器(inspector)中完成的任何配置（如果需要，还可以按场景覆盖）。在场景中添加后，您可以将其拖放到`Context`的Installers属性(Installers property)中

2. **预制件(Prefabs)**. 您也可以直接将安装程序预制件(installer prefab)从项目(Project)选项卡拖到SceneContext的InstallerPrefabs属性中。请注意，在这种情况下，您不能像在场景中实例化预制件时那样具有每场景覆盖，但可以很好地避免场景中的混乱。

3. **Resources文件夹中的预制件**.  您还可以将安装程序预制件放在Resoures文件夹下，并使用Resoures路径直接从代码中安装它们。有关使用的详细信息，请参[见此](runtime-parameters-for-installers.md)

除`MonoInstaller`和`Installer<>`之外的另一个选项是使用`ScriptableObjectInstaller`，它具有一些独特的优点（特别是对于设置(settings)） - 详情请参见[此处](scriptableobject-installer.md)。 

从其他安装程序调用安装程序时，通常需要将参数传递给它。有关如何完成的详细信息，请参见[此处](runtime-parameters-for-installer.md)。
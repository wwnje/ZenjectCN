# Hello World Example

```csharp
using Zenject;
using UnityEngine;
using System.Collections;

public class TestInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<string>().FromInstance("Hello World!");
        Container.Bind<Greeter>().AsSingle().NonLazy();
    }
}

public class Greeter
{
    public Greeter(string message)
    {
        Debug.Log(message);
    }
}
```

您可以通过执行以下操作来运行此示例：

* 在Unity中创建一个新场景
* 在Hierarchy选项卡内右键单击并选择`Zenject  - > Scene Context`
* 右键单击项目选项卡中的文件夹，然后选择`Create -> Zenject -> MonoInstaller`. 将其命名为TestInstaller.cs
* 将TestInstaller脚本添加到场景中（作为自己的GameObject或与SceneContext在同一GameObject上，无所谓）
* 通过在"Installers"属性的检查器中添加新行（按+按钮），然后将TestInstaller拖到其中，将TestInstaller的引用添加到SceneContext的属性中
* 打开TestInstaller并将上面的代码粘贴到其中
*通过选择 Edit -> Zenject -> Validate Current Scene或按`CTRL + ALT + V`来验证场景是否可以运行。 （请注意，此步骤不是必需的，但是一个良好的步骤）
* 运行工程
* 另请注意，您可以使用快捷键`CTRL + SHIFT + R`来“验证然后运行”。验证通常足够快，这不会给运行游戏增加明显的开销，并且当它确实检测到错误时，迭代起来要快得多，因为您可以避免启动时间。
* 观察console输出

SceneContext MonoBehaviour是应用程序的入口点，Zenject在开始场景之前设置所有各种依赖项。 要向Zenject场景添加内容，您需要将Zenject中引用的内容编写为“安装程序(Installer)”，它将声明所有依赖关系及其相互之间的关系。 标记为“NonLazy”的所有依赖项在安装程序运行后自动创建，这就是我们在上面添加的Greeter类在启动时创建的原因。 如果这对你没有意义，请继续阅读！